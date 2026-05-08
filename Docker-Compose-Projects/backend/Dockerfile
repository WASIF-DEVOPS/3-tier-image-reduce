# ============================================
# Stage 1: Build & Dependencies
# ============================================
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
# Install only production dependencies
RUN npm ci --only=production

# ============================================
# Stage 2: Production Image (Slim)
# ============================================
FROM node:18-alpine
WORKDIR /app

# Copy production node_modules from builder
COPY --from=builder /app/node_modules ./node_modules
COPY . .

# Run as non-root for K8s security
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

EXPOSE 3500
CMD ["node", "server.js"]