# Build Stage
FROM node:20-alpine AS build

# Set working directory for the build stage
WORKDIR /app

# Copy package.json and yarn.lock files from the root of the monorepo
COPY package.json yarn.lock ./

# Copy the backend's package.json if it's separate for Yarn workspaces
# If you have a single yarn.lock at the root managing all workspaces, these might not be strictly needed for dependency resolution,
# but it's good practice to ensure the backend's package.json is present for its own scripts.
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

# Set the working directory to the *compiled* output directory of your backend
# This is crucial so Medusa finds its compiled files and medusa-config.js
# Render's project root is /opt/render/project/src/
# Your backend code is copied to /opt/render/project/src/apps/backend
# And within that, the compiled output is in .medusa/server
WORKDIR /opt/render/project/src/apps/backend/.medusa/server

# Copy the *compiled* backend application from the build stage
# We only need the compiled output for production.
# This copies everything from /app/apps/backend/.medusa/server in build stage
# to /opt/render/project/src/apps/backend/.medusa/server (our current WORKDIR) in production stage.
COPY --from=build /app/apps/backend/.medusa/server ./

# If you have node_modules specific to the compiled output that Medusa needs,
# you might need to copy them from the build stage, but usually, Medusa bundles what it needs.
# If not, you might need to reconsider where yarn install happens or what gets copied.
# For now, let's assume `medusa build` handles dependencies or they are resolved from the root node_modules.

# Expose the port Medusa listens on
EXPOSE 9000

# Command to run the application
# Run the compiled main JavaScript file directly using node.
# Medusa will automatically look for 'medusa-config.js' in the current WORKDIR
# which is now /opt/render/project/src/apps/backend/.medusa/server
CMD ["node", "main.js"]
