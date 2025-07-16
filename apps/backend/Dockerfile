# Build Stage
FROM node:20-alpine AS build

# Set working directory for the build stage
WORKDIR /app

# Copy package.json and yarn.lock files from the root of the monorepo
# This is crucial for Turborepo and workspace setup
COPY package.json yarn.lock ./
COPY apps/backend/package.json ./apps/backend/
COPY apps/backend/yarn.lock ./apps/backend/ # Copia el lockfile específico del backend si existe

# Install global Turborepo CLI
RUN yarn add global turbo

# Install monorepo dependencies
RUN yarn install --immutable

# Copy the rest of the monorepo source code
COPY . .

# Build the backend application
# Change to the backend directory before running its build script
WORKDIR /app/apps/backend
RUN yarn build # This will run "medusa build" as per your package.json

# Production Stage
FROM node:20-alpine AS production

# Set environment variables for production
ENV NODE_ENV=production
ENV PORT=9000

# Set working directory for the production stage
# This is the key change: set WORKDIR to your backend's folder
WORKDIR /opt/render/project/src/apps/backend

# Copy the built backend from the build stage to the production image
# Ensure the entire built 'apps/backend' content is copied to the new WORKDIR
COPY --from=build /app/apps/backend ./

# Expose the port Medusa listens on
EXPOSE 9000

# Command to run the application
CMD ["medusa", "start", "--types=false"]
