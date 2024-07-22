# Aufgabe 2
### Dockerfile 1

```dockerfile
# Stage 1: Build the Next.js application
FROM node:18-alpine AS builder

# Set working directory
WORKDIR /app

# Copy package.json and package-lock.json
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the rest of the application code
COPY . .

# Build the Next.js application
RUN npm run build

# Stage 2: Run the Next.js application
FROM node:21-alpine

# Set working directory
WORKDIR /app

# Copy the build output and dependencies from the builder stage
COPY --from=builder /app ./

# Install only production dependencies
RUN npm ci --only=production

# Expose port 3000
EXPOSE 3000

# Start the Next.js application
CMD ["npm", "start"]
```

### Dockerfile 2

```dockerfile
FROM node:21 as base

WORKDIR /home/node/app

COPY package*.json ./

RUN npm i

COPY . .

EXPOSE 3000

CMD [ "npm" , "run", "dev" ]
```


#### Dockerfile 1

**Vorteile:**
1. **Mehrstufiges Build:**
   - Es verwendet ein mehrstufiges Build, wodurch unnötige Dateien und Abhängigkeiten im finalen Image vermieden werden. Dadurch wird das Image kleiner und sicherer.
2. **Build und Run Trennung:**
   - Die Trennung zwischen dem Build- und dem Run-Image sorgt für eine klare Aufteilung und ermöglicht die Verwendung von unterschiedlichen Node.js-Versionen für Build und Run, was Flexibilität bietet.
3. **Produktion-Abhängigkeiten:**
   - Im finalen Image werden nur Produktionsabhängigkeiten installiert (`npm ci --only=production`), was die Größe des Images weiter reduziert und das Sicherheitsrisiko verringert.

**Nachteile:**
1. **Komplexität:**
   - Die mehrstufige Struktur und der zusätzliche Schritt zum Installieren der Produktionsabhängigkeiten machen das Dockerfile etwas komplexer.

#### Dockerfile 2

**Vorteile:**
1. **Einfachheit:**
   - Das Dockerfile ist einfacher und enthält weniger Schritte, was es leichter verständlich macht.

**Nachteile:**
1. **Größeres Image:**
   - Da keine Trennung zwischen Build- und Run-Phasen besteht, enthält das finale Image sowohl Entwicklungs- als auch Produktionsabhängigkeiten, was die Größe des Images unnötig erhöht.
2. **Entwicklungsmodus:**
   - Es wird `npm run dev` zum Starten verwendet, was normalerweise für die Entwicklungsumgebung vorgesehen ist und nicht für die Produktion. In der Produktion sollte `npm start` Befehl verwendet werden, der das optimierte Build verwendet.
3. **Sicherheitsrisiko:**
   - Die Installation von allen Abhängigkeiten (inklusive Entwicklungsabhängigkeiten) und das Kopieren aller Dateien in das finale Image erhöht das Sicherheitsrisiko.

### Bewertung und Empfehlung

**Dockerfile 1** ist besser für das Deployment meiner Frontend-App geeignet, und zwar aus folgenden Gründen:

- **Effizienz:** Das mehrstufige Build sorgt dafür, dass nur notwendige Dateien und Produktionsabhängigkeiten in das finale Image gelangen, was die Größe reduziert und das Image effizienter macht.
- **Sicherheit:** Durch die Trennung der Build- und Run-Phasen sowie die Installation von nur Produktionsabhängigkeiten wird die Angriffsfläche minimiert.
- **Produktionstauglichkeit:** Es verwendet `npm start` für den Start, was für eine Produktionsumgebung geeignet ist.

**Dockerfile 1** ist besser für das Deployment meiner Frontend-App geeignet, da es ein mehrstufiges Build verwendet, nur Produktionsabhängigkeiten installiert und damit ein kleineres, sichereres und effizienteres Image erstellt.

---

Der Unterschied im Entwicklungsmodus liegt hauptsächlich in der Art und Weise, wie die Anwendung läuft und welche Abhängigkeiten installiert sind:

1. **Laufmodus:**
   - **Entwicklungsmodus (`npm run dev`):**
     - Die Anwendung läuft in einem Modus, der Hot-Reloading unterstützt, sodass Änderungen am Code sofort reflektiert werden, ohne dass der Server neu gestartet werden muss.
     - Entwicklertools und Debugging-Funktionen sind aktiviert, was die Fehlersuche erleichtert.
   - **Produktionsmodus (`npm start`):**
     - Die Anwendung läuft in einem optimierten Modus, der für Performance und Stabilität in der Produktionsumgebung ausgelegt ist.
     - Es werden keine Entwicklertools oder Debugging-Funktionen geladen, wodurch die Anwendung schneller und sicherer ist.

2. **Abhängigkeiten:**
   - **Entwicklungsmodus:**
     - Es werden alle Abhängigkeiten installiert, einschließlich Entwicklungsabhängigkeiten (devDependencies) wie Linter, Testbibliotheken und andere Tools, die nur während der Entwicklung benötigt werden.
   - **Produktionsmodus:**
     - Es werden nur Produktionsabhängigkeiten installiert, die für den Betrieb der Anwendung erforderlich sind, was das Image schlanker und sicherer macht.

3. **Bildgröße und Sicherheit:**
   - **Entwicklungsmodus:**
     - Größeres Image, da alle Abhängigkeiten installiert sind und der gesamte Quellcode sowie Entwicklertools enthalten sind.
     - Höhere Angriffsfläche durch zusätzliche Abhängigkeiten und Tools.
   - **Produktionsmodus:**
     - Kleineres Image, da nur notwendige Dateien und Abhängigkeiten enthalten sind.
     - Geringere Angriffsfläche, da unnötige Tools und Abhängigkeiten entfernt sind.

Durch die Trennung von Entwicklungs- und Produktionsmodus sowie die Nutzung eines mehrstufigen Builds in Dockerfile 1 wird sichergestellt, dass die Anwendung in der Produktionsumgebung sicher, effizient und performant läuft.