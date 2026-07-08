# Stage 1: Dependencies
FROM node:22-alpine AS deps
WORKDIR /app

# Install pnpm and required OpenSSL dependencies
RUN apk add --no-cache openssl openssl-dev
RUN npm install -g pnpm@9

# Copy package.json and pnpm-lock.yaml
COPY package.json pnpm-lock.yaml ./

# Install dependencies
RUN pnpm install --frozen-lockfile

# Stage 2: Builder
FROM node:22-alpine AS builder
WORKDIR /app

# Install pnpm and required OpenSSL dependencies
RUN apk add --no-cache openssl openssl-dev
RUN npm install -g pnpm@9

# Copy dependencies from deps stage
COPY --from=deps /app/node_modules ./node_modules

# Copy all files
COPY . .

# Set environment variables for building
ENV NEXT_TELEMETRY_DISABLED 1

ARG NEXT_PUBLIC_API_BASE_URL
ARG NEXT_PUBLIC_SITE_URL
ARG NEXT_PUBLIC_TURNSTILE_SITE_KEY
ARG NEXT_PUBLIC_GOOGLE_CLIENT_ID
ARG NEXT_PUBLIC_TAWKTO_EMBED_URL

# Re-export as ENV so `next build` can actually read them
ENV NEXT_PUBLIC_API_BASE_URL=$NEXT_PUBLIC_API_BASE_URL
ENV NEXT_PUBLIC_SITE_URL=$NEXT_PUBLIC_SITE_URL
ENV NEXT_PUBLIC_TURNSTILE_SITE_KEY=$NEXT_PUBLIC_TURNSTILE_SITE_KEY
ENV NEXT_PUBLIC_GOOGLE_CLIENT_ID=$NEXT_PUBLIC_GOOGLE_CLIENT_ID
ENV NEXT_PUBLIC_TAWKTO_EMBED_URL=$NEXT_PUBLIC_TAWKTO_EMBED_URL

# Build the application
RUN pnpm build

# Stage 3: Production runner
FROM node:22-alpine AS runner
WORKDIR /app

# Install pnpm and required OpenSSL dependencies
RUN apk add --no-cache openssl openssl-dev
RUN npm install -g pnpm@9

# Set environment variables
ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1

# Add a non-root user to run the app
RUN addgroup --system --gid 1001 nodejs && \
    adduser --system --uid 1001 nextjs && \
    chown -R nextjs:nodejs /app

# Copy necessary files for the standalone app
COPY --from=builder --chown=nextjs:nodejs /app/next.config.js ./
COPY --from=builder --chown=nextjs:nodejs /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static



# Switch to non-root user
USER nextjs

# Expose the port the app will run on
ENV PORT 3000
EXPOSE ${PORT}
ENV HOSTNAME "0.0.0.0"



# Health check to verify app is running
# HEALTHCHECK --interval=30s --timeout=30s --start-period=5s --retries=3 \
#   CMD wget -q -O - http://localhost:${PORT}/api/health || exit 1

CMD ["node", "server"] 
