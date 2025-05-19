# Use the official Go image as a base
FROM golang:1.21-alpine

# Set the working directory
WORKDIR /app

# Copy the Go app source
COPY . .

# Build the Go app
RUN go mod init netauto.com/m
RUN go build -o server

# Set environment variables (can be overridden at runtime)
ENV NAME=Alice
ENV AGE=25

# Expose the port the app runs on
EXPOSE 8080

# Command to run the executable
CMD ["./server"]
