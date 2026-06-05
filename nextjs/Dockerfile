# Stage 1: Build
FROM node:24-alpine AS builder
RUN npm config set registry https://mirror-npm.runflare.com

WORKDIR /app

# Copy package files
COPY package.json ./

# Install dependencies
RUN npm install

# Copy source code
COPY . .

ENV NEXT_PUBLIC_LIVEKIT_URL=wss://golden-horde.ir

# Build Next.js for production
RUN npm run build

# Stage 2: Production image
FROM node:24-alpine AS runner

RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

# Copy only what is needed
COPY --from=builder --chown=appuser:appgroup /app/package.json ./
COPY --from=builder --chown=appuser:appgroup /app/package-lock.json ./
COPY --from=builder --chown=appuser:appgroup /app/node_modules ./node_modules
COPY --from=builder --chown=appuser:appgroup /app/.next ./.next
COPY --from=builder --chown=appuser:appgroup /app/public ./public

USER appuser

# Environment
ENV NODE_ENV=production

# Expose port for Traefik network (internal only)
EXPOSE 3000

# Start production
CMD ["sh", "-c", "export LIVEKIT_API_KEY=$(cat $LIVEKIT_API_KEY_FILE) && export LIVEKIT_API_SECRET=$(cat $LIVEKIT_API_SECRET_FILE) && npm run start"]
