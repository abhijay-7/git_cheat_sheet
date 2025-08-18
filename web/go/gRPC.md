# Proto3 & gRPC in Go - Cheatsheet

## Table of Contents
1. [Proto3 Basics](#proto3-basics)
2. [Data Types](#data-types)
3. [Message Definition](#message-definition)
4. [Service Definition](#service-definition)
5. [Code Generation](#code-generation)
6. [gRPC Server Setup](#grpc-server-setup)
7. [gRPC Client Setup](#grpc-client-setup)
8. [Streaming](#streaming)
9. [Error Handling](#error-handling)
10. [Middleware/Interceptors](#middlewareinterceptors)
11. [Common Patterns](#common-patterns)

## Proto3 Basics

### Basic Syntax
```protobuf
syntax = "proto3";

package mypackage;
option go_package = "github.com/user/repo/pb";

import "google/protobuf/timestamp.proto";
```

### Field Rules
- **singular**: Default, field can have 0 or 1 value
- **repeated**: Field can have 0 or more values (like arrays)
- **optional**: Explicitly optional (proto3.15+)

## Data Types

### Scalar Types
| Proto Type | Go Type | Notes |
|------------|---------|-------|
| `double` | `float64` | |
| `float` | `float32` | |
| `int32` | `int32` | Variable-length encoding |
| `int64` | `int64` | Variable-length encoding |
| `uint32` | `uint32` | Variable-length encoding |
| `uint64` | `uint64` | Variable-length encoding |
| `sint32` | `int32` | Efficient for negative numbers |
| `sint64` | `int64` | Efficient for negative numbers |
| `fixed32` | `uint32` | Always 4 bytes |
| `fixed64` | `uint64` | Always 8 bytes |
| `sfixed32` | `int32` | Always 4 bytes |
| `sfixed64` | `int64` | Always 8 bytes |
| `bool` | `bool` | |
| `string` | `string` | UTF-8 encoded |
| `bytes` | `[]byte` | Arbitrary byte sequence |

### Well-Known Types
```protobuf
import "google/protobuf/timestamp.proto";
import "google/protobuf/duration.proto";
import "google/protobuf/empty.proto";
import "google/protobuf/wrappers.proto";

google.protobuf.Timestamp created_at = 1;
google.protobuf.Duration timeout = 2;
google.protobuf.Empty empty = 3;
google.protobuf.StringValue optional_string = 4;
```

## Message Definition

### Basic Message
```protobuf
message User {
  int64 id = 1;
  string name = 2;
  string email = 3;
  repeated string tags = 4;
  Address address = 5;
}

message Address {
  string street = 1;
  string city = 2;
  string country = 3;
}
```

### Enums
```protobuf
enum Status {
  STATUS_UNSPECIFIED = 0;  // Always have a zero value
  STATUS_ACTIVE = 1;
  STATUS_INACTIVE = 2;
  STATUS_PENDING = 3;
}

message User {
  Status status = 1;
}
```

### Oneof
```protobuf
message SearchRequest {
  oneof query {
    string text = 1;
    int32 page_number = 2;
  }
}
```

### Maps
```protobuf
message User {
  map<string, string> metadata = 1;
  map<int32, Address> addresses = 2;
}
```

## Service Definition

### Basic Service
```protobuf
service UserService {
  rpc GetUser(GetUserRequest) returns (GetUserResponse);
  rpc ListUsers(ListUsersRequest) returns (ListUsersResponse);
  rpc CreateUser(CreateUserRequest) returns (CreateUserResponse);
  rpc UpdateUser(UpdateUserRequest) returns (UpdateUserResponse);
  rpc DeleteUser(DeleteUserRequest) returns (google.protobuf.Empty);
}
```

### Streaming Services
```protobuf
service StreamService {
  // Server streaming
  rpc ListUsers(ListUsersRequest) returns (stream User);
  
  // Client streaming
  rpc CreateUsers(stream CreateUserRequest) returns (CreateUsersResponse);
  
  // Bidirectional streaming
  rpc Chat(stream ChatMessage) returns (stream ChatMessage);
}
```

## Code Generation

### Installation
```bash
# Install protoc compiler
# macOS
brew install protobuf

# Ubuntu/Debian
sudo apt install protobuf-compiler

# Install Go plugins
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```

### Generate Code
```bash
# Basic generation
protoc --go_out=. --go-grpc_out=. proto/*.proto

# With custom paths
protoc --go_out=./pb --go-grpc_out=./pb \
  --go_opt=paths=source_relative \
  --go-grpc_opt=paths=source_relative \
  proto/*.proto
```

### Makefile Example
```makefile
.PHONY: proto
proto:
	protoc --go_out=./pb --go-grpc_out=./pb \
		--go_opt=paths=source_relative \
		--go-grpc_opt=paths=source_relative \
		proto/*.proto
```

## gRPC Server Setup

### Basic Server
```go
package main

import (
    "context"
    "log"
    "net"

    "google.golang.org/grpc"
    pb "your-module/pb"
)

type server struct {
    pb.UnimplementedUserServiceServer
}

func (s *server) GetUser(ctx context.Context, req *pb.GetUserRequest) (*pb.GetUserResponse, error) {
    return &pb.GetUserResponse{
        User: &pb.User{
            Id:   req.Id,
            Name: "John Doe",
            Email: "john@example.com",
        },
    }, nil
}

func main() {
    lis, err := net.Listen("tcp", ":50051")
    if err != nil {
        log.Fatalf("failed to listen: %v", err)
    }

    s := grpc.NewServer()
    pb.RegisterUserServiceServer(s, &server{})

    log.Println("Server listening on :50051")
    if err := s.Serve(lis); err != nil {
        log.Fatalf("failed to serve: %v", err)
    }
}
```

### Server with Options
```go
func main() {
    opts := []grpc.ServerOption{
        grpc.MaxRecvMsgSize(1024 * 1024 * 4), // 4MB
        grpc.MaxSendMsgSize(1024 * 1024 * 4), // 4MB
        grpc.ConnectionTimeout(time.Second * 5),
        grpc.UnaryInterceptor(loggingInterceptor),
    }
    
    s := grpc.NewServer(opts...)
    pb.RegisterUserServiceServer(s, &server{})
    
    // ... rest of setup
}
```

## gRPC Client Setup

### Basic Client
```go
package main

import (
    "context"
    "log"
    "time"

    "google.golang.org/grpc"
    "google.golang.org/grpc/credentials/insecure"
    pb "your-module/pb"
)

func main() {
    conn, err := grpc.Dial("localhost:50051", 
        grpc.WithTransportCredentials(insecure.NewCredentials()))
    if err != nil {
        log.Fatalf("did not connect: %v", err)
    }
    defer conn.Close()

    client := pb.NewUserServiceClient(conn)

    ctx, cancel := context.WithTimeout(context.Background(), time.Second)
    defer cancel()

    resp, err := client.GetUser(ctx, &pb.GetUserRequest{Id: 1})
    if err != nil {
        log.Fatalf("could not get user: %v", err)
    }

    log.Printf("User: %v", resp.User)
}
```

### Client with Options
```go
func createClient() pb.UserServiceClient {
    opts := []grpc.DialOption{
        grpc.WithTransportCredentials(insecure.NewCredentials()),
        grpc.WithTimeout(time.Second * 5),
        grpc.WithBlock(),
        grpc.WithUnaryInterceptor(clientLoggingInterceptor),
    }
    
    conn, err := grpc.Dial("localhost:50051", opts...)
    if err != nil {
        log.Fatalf("Failed to connect: %v", err)
    }
    
    return pb.NewUserServiceClient(conn)
}
```

## Streaming

### Server Streaming
```go
// Server side
func (s *server) ListUsers(req *pb.ListUsersRequest, stream pb.UserService_ListUsersServer) error {
    for i := 1; i <= 10; i++ {
        user := &pb.User{
            Id:   int64(i),
            Name: fmt.Sprintf("User %d", i),
        }
        
        if err := stream.Send(&pb.User{User: user}); err != nil {
            return err
        }
        
        time.Sleep(time.Second) // Simulate delay
    }
    return nil
}

// Client side
func listUsers(client pb.UserServiceClient) {
    stream, err := client.ListUsers(context.Background(), &pb.ListUsersRequest{})
    if err != nil {
        log.Fatalf("Error calling ListUsers: %v", err)
    }
    
    for {
        resp, err := stream.Recv()
        if err == io.EOF {
            break
        }
        if err != nil {
            log.Fatalf("Error receiving: %v", err)
        }
        
        log.Printf("Received user: %v", resp.User)
    }
}
```

### Client Streaming
```go
// Server side
func (s *server) CreateUsers(stream pb.UserService_CreateUsersServer) error {
    var count int32
    
    for {
        req, err := stream.Recv()
        if err == io.EOF {
            return stream.SendAndClose(&pb.CreateUsersResponse{
                Count: count,
            })
        }
        if err != nil {
            return err
        }
        
        // Process user creation
        log.Printf("Creating user: %v", req.User)
        count++
    }
}

// Client side
func createUsers(client pb.UserServiceClient) {
    stream, err := client.CreateUsers(context.Background())
    if err != nil {
        log.Fatalf("Error calling CreateUsers: %v", err)
    }
    
    users := []*pb.User{
        {Name: "Alice", Email: "alice@example.com"},
        {Name: "Bob", Email: "bob@example.com"},
    }
    
    for _, user := range users {
        if err := stream.Send(&pb.CreateUserRequest{User: user}); err != nil {
            log.Fatalf("Error sending: %v", err)
        }
    }
    
    resp, err := stream.CloseAndRecv()
    if err != nil {
        log.Fatalf("Error closing: %v", err)
    }
    
    log.Printf("Created %d users", resp.Count)
}
```

### Bidirectional Streaming
```go
// Server side
func (s *server) Chat(stream pb.UserService_ChatServer) error {
    for {
        msg, err := stream.Recv()
        if err == io.EOF {
            return nil
        }
        if err != nil {
            return err
        }
        
        // Echo the message back
        response := &pb.ChatMessage{
            Message: fmt.Sprintf("Echo: %s", msg.Message),
            User:    msg.User,
        }
        
        if err := stream.Send(response); err != nil {
            return err
        }
    }
}

// Client side
func chat(client pb.UserServiceClient) {
    stream, err := client.Chat(context.Background())
    if err != nil {
        log.Fatalf("Error calling Chat: %v", err)
    }
    
    go func() {
        for {
            resp, err := stream.Recv()
            if err == io.EOF {
                return
            }
            if err != nil {
                log.Fatalf("Error receiving: %v", err)
            }
            log.Printf("Received: %s", resp.Message)
        }
    }()
    
    messages := []string{"Hello", "How are you?", "Goodbye"}
    for _, msg := range messages {
        if err := stream.Send(&pb.ChatMessage{
            Message: msg,
            User:    "client",
        }); err != nil {
            log.Fatalf("Error sending: %v", err)
        }
        time.Sleep(time.Second)
    }
    
    stream.CloseSend()
}
```

## Error Handling

### Standard Error Codes
```go
import (
    "google.golang.org/grpc/codes"
    "google.golang.org/grpc/status"
)

// Return error with code
func (s *server) GetUser(ctx context.Context, req *pb.GetUserRequest) (*pb.GetUserResponse, error) {
    if req.Id <= 0 {
        return nil, status.Errorf(codes.InvalidArgument, "user ID must be positive")
    }
    
    user, err := s.userRepo.GetUser(req.Id)
    if err != nil {
        if errors.Is(err, ErrUserNotFound) {
            return nil, status.Errorf(codes.NotFound, "user not found")
        }
        return nil, status.Errorf(codes.Internal, "internal server error")
    }
    
    return &pb.GetUserResponse{User: user}, nil
}

// Client error handling
resp, err := client.GetUser(ctx, req)
if err != nil {
    if st, ok := status.FromError(err); ok {
        switch st.Code() {
        case codes.NotFound:
            log.Println("User not found")
        case codes.InvalidArgument:
            log.Println("Invalid argument:", st.Message())
        default:
            log.Printf("Error: %v", err)
        }
    }
    return
}
```

### Rich Error Details
```go
import (
    "google.golang.org/genproto/googleapis/rpc/errdetails"
    "google.golang.org/grpc/status"
)

// Server side - adding error details
func (s *server) CreateUser(ctx context.Context, req *pb.CreateUserRequest) (*pb.CreateUserResponse, error) {
    if req.User.Email == "" {
        st := status.New(codes.InvalidArgument, "validation failed")
        
        v := &errdetails.BadRequest_FieldViolation{
            Field:       "email",
            Description: "email is required",
        }
        
        br := &errdetails.BadRequest{
            FieldViolations: []*errdetails.BadRequest_FieldViolation{v},
        }
        
        st, _ = st.WithDetails(br)
        return nil, st.Err()
    }
    
    // ... rest of implementation
}

// Client side - extracting error details
_, err := client.CreateUser(ctx, req)
if err != nil {
    st := status.Convert(err)
    for _, detail := range st.Details() {
        switch t := detail.(type) {
        case *errdetails.BadRequest:
            for _, violation := range t.GetFieldViolations() {
                log.Printf("Field: %s, Error: %s", violation.GetField(), violation.GetDescription())
            }
        }
    }
}
```

## Middleware/Interceptors

### Unary Server Interceptor
```go
func loggingInterceptor(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo, handler grpc.UnaryHandler) (interface{}, error) {
    start := time.Now()
    
    log.Printf("Starting request: %s", info.FullMethod)
    
    resp, err := handler(ctx, req)
    
    log.Printf("Completed request: %s, Duration: %v, Error: %v", 
        info.FullMethod, time.Since(start), err)
    
    return resp, err
}
```

### Stream Server Interceptor
```go
func streamLoggingInterceptor(srv interface{}, stream grpc.ServerStream, info *grpc.StreamServerInfo, handler grpc.StreamHandler) error {
    log.Printf("Starting stream: %s", info.FullMethod)
    
    err := handler(srv, stream)
    
    log.Printf("Completed stream: %s, Error: %v", info.FullMethod, err)
    
    return err
}
```

### Client Interceptor
```go
func clientLoggingInterceptor(ctx context.Context, method string, req, reply interface{}, cc *grpc.ClientConn, invoker grpc.UnaryInvoker, opts ...grpc.CallOption) error {
    start := time.Now()
    
    err := invoker(ctx, method, req, reply, cc, opts...)
    
    log.Printf("Client call: %s, Duration: %v, Error: %v", 
        method, time.Since(start), err)
    
    return err
}
```

## Common Patterns

### Health Checking
```go
import "google.golang.org/grpc/health/grpc_health_v1"

type healthServer struct {
    grpc_health_v1.UnimplementedHealthServer
}

func (h *healthServer) Check(ctx context.Context, req *grpc_health_v1.HealthCheckRequest) (*grpc_health_v1.HealthCheckResponse, error) {
    return &grpc_health_v1.HealthCheckResponse{
        Status: grpc_health_v1.HealthCheckResponse_SERVING,
    }, nil
}

// Register health server
grpc_health_v1.RegisterHealthServer(s, &healthServer{})
```

### Context with Metadata
```go
import (
    "google.golang.org/grpc/metadata"
)

// Server - reading metadata
func (s *server) GetUser(ctx context.Context, req *pb.GetUserRequest) (*pb.GetUserResponse, error) {
    md, ok := metadata.FromIncomingContext(ctx)
    if !ok {
        return nil, status.Errorf(codes.InvalidArgument, "missing metadata")
    }
    
    userID := md.Get("user-id")
    log.Printf("Request from user: %v", userID)
    
    // ... rest of implementation
}

// Client - sending metadata
func callWithMetadata(client pb.UserServiceClient) {
    md := metadata.Pairs("user-id", "12345")
    ctx := metadata.NewOutgoingContext(context.Background(), md)
    
    resp, err := client.GetUser(ctx, &pb.GetUserRequest{Id: 1})
    // ... handle response
}
```

### Graceful Shutdown
```go
func main() {
    lis, err := net.Listen("tcp", ":50051")
    if err != nil {
        log.Fatalf("failed to listen: %v", err)
    }

    s := grpc.NewServer()
    pb.RegisterUserServiceServer(s, &server{})

    go func() {
        log.Println("Starting server on :50051")
        if err := s.Serve(lis); err != nil {
            log.Fatalf("failed to serve: %v", err)
        }
    }()

    // Wait for interrupt signal
    c := make(chan os.Signal, 1)
    signal.Notify(c, os.Interrupt, syscall.SIGTERM)
    <-c

    log.Println("Shutting down server...")
    s.GracefulStop()
}
```

### Connection Pooling (Client)
```go
type ClientPool struct {
    connections []*grpc.ClientConn
    clients     []pb.UserServiceClient
    current     int32
    mu          sync.RWMutex
}

func NewClientPool(address string, size int) (*ClientPool, error) {
    pool := &ClientPool{
        connections: make([]*grpc.ClientConn, size),
        clients:     make([]pb.UserServiceClient, size),
    }
    
    for i := 0; i < size; i++ {
        conn, err := grpc.Dial(address, grpc.WithTransportCredentials(insecure.NewCredentials()))
        if err != nil {
            return nil, err
        }
        
        pool.connections[i] = conn
        pool.clients[i] = pb.NewUserServiceClient(conn)
    }
    
    return pool, nil
}

func (p *ClientPool) GetClient() pb.UserServiceClient {
    p.mu.RLock()
    defer p.mu.RUnlock()
    
    idx := atomic.AddInt32(&p.current, 1) % int32(len(p.clients))
    return p.clients[idx]
}

func (p *ClientPool) Close() {
    for _, conn := range p.connections {
        conn.Close()
    }
}
```

## Useful Commands

```bash
# List all services in a proto file
protoc --descriptor_set_out=/dev/stdout --include_imports proto/*.proto | \
  grpc_cli ls /dev/stdin

# Test gRPC service
grpc_cli call localhost:50051 UserService.GetUser "id: 1"

# Generate documentation
protoc --doc_out=./docs --doc_opt=html,index.html proto/*.proto
```

## Dependencies (go.mod)
```go
require (
    google.golang.org/grpc v1.58.0
    google.golang.org/protobuf v1.31.0
    google.golang.org/genproto/googleapis/rpc v0.0.0-20230913181813-007df8e322eb
)
```