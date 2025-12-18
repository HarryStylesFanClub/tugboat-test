# Stage 1: Build stage
# Use golang:alpine as the build image - it includes Go compiler and build tools
# Alpine is based on musl libc and is much smaller than Debian-based images
FROM golang:alpine AS builder

# Set the working directory inside the container
WORKDIR /app

# Copy go mod files first (for better Docker layer caching)
# This allows Docker to cache dependencies if they haven't changed
COPY go.mod go.sum ./

# Download dependencies (this layer will be cached if go.mod/go.sum don't change)
RUN go mod download

# Copy the rest of the source code
COPY . .

# Build the Go application
# CGO_ENABLED=0 creates a statically linked binary (no C dependencies)
# -ldflags="-s -w" strips debug info to reduce binary size
# -o server specifies the output binary name
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o server .

# Stage 2: Runtime stage
# Use alpine:latest as the final image - it's only ~5MB
# This is the barebone image that will run your server
FROM alpine:latest

# Install ca-certificates for HTTPS connections (needed if your app makes external HTTPS calls)
# This is optional but recommended for production
RUN apk --no-cache add ca-certificates

# Create a non-root user for security
RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser

# Set the working directory
WORKDIR /app

# Copy the compiled binary from the builder stage
COPY --from=builder /app/server .

# Copy the public directory (static files)
COPY --from=builder /app/public ./public

# Change ownership to the non-root user
RUN chown -R appuser:appuser /app

# Switch to non-root user
USER appuser

# Expose port 8080 (the port your server listens on)
EXPOSE 8080

# Run the server binary
CMD ["./server"]

