# Build Stage
FROM node:20-alpine AS build

# Set working directory for the build stage
WORKDIR /app

# Copy package.json and yarn.lock files from the root of the monorepo
COPY package.json yarn.lock ./
COPY apps/backend/package.json ./apps/backend/
# If apps/backend has its own yarn.lock, copy it too
COPY apps/backend/yarn.lock ./apps/backend/

# Install global Turborepo CLI
RUN yarn add global turbo

# Install monorepo dependencies
RUN yarn install --immutable

# Copy the rest of the monorepo source code
COPY . .

# Build the backend application
WORKDIR /app/apps/backend
RUN yarn build # This will run "medusa build" as per your package.json

# Production Stage
FROM node:20-alpine AS production

# Set environment variables for production
ENV NODE_ENV=production
ENV PORT=9000

# Set working directory for the production stage
# We still want this here for relative paths within the backend, but it seems Medusa CLI has its own logic
WORKDIR /opt/render/project/src/apps/backend

# Copy the built backend from the build stage
COPY --from=build /app/apps/backend ./

# !!! ESTE ES EL CAMBIO CLAVE PARA EL MEDUSA-CONFIG !!!
# Copia medusa-config.ts a la raíz del directorio del proyecto Render
# Render copia la raíz de tu repo a /opt/render/project/src/
# Entonces, queremos que medusa-config.ts esté en /opt/render/project/src/medusa-config.ts
COPY --from=build /app/apps/backend/medusa-config.ts /opt/render/project/src/medusa-config.ts

# Expose the port Medusa listens on
EXPOSE 9000

# Command to run the application
# Ahora, Medusa debería encontrar la configuración en /opt/render/project/src/medusa-config.ts
# No es necesario el flag --config si el archivo está en la ubicación predeterminada.
CMD ["medusa", "start", "--types=false"]
