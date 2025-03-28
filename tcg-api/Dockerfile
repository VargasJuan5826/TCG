# Etapa de construcción usando Go 1.24
FROM golang:1.24 as builder

# Set the working directory inside the container
WORKDIR /app

# Copy go.mod and go.sum (if they exist) to leverage Docker cache
COPY go.mod go.sum* ./

# Initialize Go module if go.mod doesn't exist
RUN if [ ! -f go.mod ]; then go mod init tcg-api; fi

# Add initial dependencies and download them
RUN go get github.com/99designs/gqlgen && \
    go get github.com/go-sql-driver/mysql

# Copy the rest of the application code, including schema.graphqls and gqlgen.yml
COPY . .

# Ensure all dependencies (including transitive ones) are resolved for gqlgen
RUN go get github.com/99designs/gqlgen/codegen/config && \
    go get github.com/99designs/gqlgen/internal/imports && \
    go get github.com/99designs/gqlgen/codegen && \
    go get github.com/99designs/gqlgen && \
    go mod tidy

# Generate GraphQL code if schema.graphqls exists
RUN if [ -f graph/schema.graphqls ]; then go run github.com/99designs/gqlgen generate; fi

# Build the application with CGO disabled for a static binary
RUN CGO_ENABLED=0 GOOS=linux go build -o tcg-api ./cmd/main.go

# Etapa final usando una imagen ligera de Alpine
FROM alpine:latest

# Add CA certificates for secure connections
RUN apk --no-cache add ca-certificates

# Set the working directory in the final image
WORKDIR /root/

# Copy the compiled binary from the builder stage
COPY --from=builder /app/tcg-api .

# Expose port 8080 for the API
EXPOSE 8080

# Command to run the API
CMD ["./tcg-api"]