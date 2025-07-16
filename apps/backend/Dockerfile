# Build Stage
FROM node:20-alpine AS build

# Set working directory for the build stage
WORKDIR /app

# Copy package.json and yarn.lock files from the root of the monorepo
COPY package.json yarn.lock ./

# Copy the backend's package.json if it's separate for Yarn workspaces
COPY apps/backend/package.json ./apps/backend/

# Install global Turborepo CLI (if needed for build step, otherwise remove)
RUN yarn add global turbo

# Install monorepo dependencies
RUN yarn install --immutable

# Copy the rest of the monorepo source code
COPY . .

# Build the backend application
# This assumes 'yarn build' in /app/apps/backend runs 'medusa build'
# and that 'medusa build' compiles to /app/apps/backend/.medusa/server
WORKDIR /app/apps/backend
RUN yarn build

# Production Stage
FROM node:20-alpine AS production

# Set environment variables for production
ENV NODE_ENV=production
ENV PORT=9000

# Set the primary working directory for the application.
# This should be the root of your *copied backend application* within the Render environment.
WORKDIR /opt/render/project/src/apps/backend

# Copy the *entire built backend* from the build stage to the production stage.
# This ensures all necessary files, including the compiled .medusa/server directory, are present.
COPY --from=build /app/apps/backend ./

# Expose the port Medusa listens on
EXPOSE 9000

# Command to run the application
# We explicitly tell Node where to find the main.js file,
# combining the WORKDIR with the known compiled output path.
# This path is relative to the WORKDIR: /opt/render/project/src/apps/backend
CMD ["node", "./.medusa/server/main.js"]
