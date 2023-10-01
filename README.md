# grpc_tutorial

### In this tutorial, we build a basic API using gRPC and protobufs in Go

## Run `go run main.go` to run the app, run `go build main.go` to build an executable file.

### Check out the Youtube Tutorial for this [Go Program](https://youtu.be/Y92WWaZJl24). Here is our [Youtube Channel](https://www.youtube.com/channel/UCYqCZOwHbnPwyjawKfE21wg) Subscribe for more content.

### Check out our blog at [tensor-programming.com](http://tensor-programming.com/).

### Our [Twitter](https://twitter.com/TensorProgram), our [facebook](https://www.facebook.com/Tensor-Programming-1197847143611799/) and our [Steemit](https://steemit.com/@tensor).

Steps:
1. mkdir proto
2. touch service.proto
3. vim service.proto
4. add these lines
syntax = "proto3";

package proto;

message Request {
  int64 a = 1;
  int64 b = 2;
}

message Response { 
  int64 result = 1; 
}

service AddService {
  rpc Add(Request) returns (Response);
  rpc Multiply(Request) returns (Response);
}


5. cd ../
6. pwd
/home/soumen/lab/go-learn/test_grpc/grpc_tutorial
7. protoc --proto_path=proto --go_out=plugins=grpc:proto service.proto
8. ls -ltr proto/
    soumen@UB:~/lab/go-learn/test_grpc/grpc_tutorial$ ll proto/
    total 16
    -rw-rw-r-- 1 soumen soumen  237 Sep 30 14:36 service.proto
    -rw-rw-r-- 1 soumen soumen 8245 Sep 30 15:10 service.pb.go
9. mkdir server:
10. cd server:
    touch main.go
    vim main.go
    add these lines
    ```package main

    import (
        "context"
        "grpc_tutorial/proto"
        "net"

        "google.golang.org/grpc"
        "google.golang.org/grpc/reflection"
    )

    type server struct{}

    func main() {
        listener, err := net.Listen("tcp", ":4040")
        if err != nil {
            panic(err)
        }

        srv := grpc.NewServer()
        proto.RegisterAddServiceServer(srv, &server{})
        reflection.Register(srv)

        if e := srv.Serve(listener); e != nil {
            panic(e)
        }

    }

    func (s *server) Add(ctx context.Context, request *proto.Request) (*proto.Response, error) {
        a, b := request.GetA(), request.GetB()

        result := a + b

        return &proto.Response{Result: result}, nil
    }

    func (s *server) Multiply(ctx context.Context, request *proto.Request) (*proto.Response, error) {
        a, b := request.GetA(), request.GetB()

        result := a * b

        return &proto.Response{Result: result}, nil
    }```
11. cd ../
12. mkdir client
13. cd client
14. vim main.go
15. add these lines
        ```package main

        import (
            "fmt"
            "grpc_tutorial/proto"
            "log"
            "net/http"
            "strconv"

            "github.com/gin-gonic/gin"
            "google.golang.org/grpc"
        )

        func main() {
            conn, err := grpc.Dial("localhost:4040", grpc.WithInsecure())
            if err != nil {
                panic(err)
            }

            client := proto.NewAddServiceClient(conn)

            g := gin.Default()

            g.GET("/add/:a/:b", func(ctx *gin.Context) {
                a, err := strconv.ParseUint(ctx.Param("a"), 10, 64)
                if err != nil {
                    ctx.JSON(http.StatusBadRequest, gin.H{"error": "Invalid Parameter A"})
                    return
                }

                b, err := strconv.ParseUint(ctx.Param("b"), 10, 64)
                if err != nil {
                    ctx.JSON(http.StatusBadRequest, gin.H{"error": "Invalid Parameter B"})
                    return
                }

                req := &proto.Request{A: int64(a), B: int64(b)}
                if response, err := client.Add(ctx, req); err == nil {
                    ctx.JSON(http.StatusOK, gin.H{
                        "result": fmt.Sprint(response.Result),
                    })
                } else {
                    ctx.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
                }
            })

            g.GET("/mult/:a/:b", func(ctx *gin.Context) {
                a, err := strconv.ParseUint(ctx.Param("a"), 10, 64)
                if err != nil {
                    ctx.JSON(http.StatusBadRequest, gin.H{"error": "Invalid Parameter A"})
                    return
                }
                b, err := strconv.ParseUint(ctx.Param("b"), 10, 64)
                if err != nil {
                    ctx.JSON(http.StatusBadRequest, gin.H{"error": "Invalid Parameter B"})
                    return
                }
                req := &proto.Request{A: int64(a), B: int64(b)}

                if response, err := client.Multiply(ctx, req); err == nil {
                    ctx.JSON(http.StatusOK, gin.H{
                        "result": fmt.Sprint(response.Result),
                    })
                } else {
                    ctx.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
                }
            })

            if err := g.Run(":8080"); err != nil {
                log.Fatalf("Failed to run server: %v", err)
            }

        }```
16. cd ../
17. pwd
18. mod init example.com/packages
19. go mod tidy
20. cd server; go run main.go
21. open different terminal and cd to client and run
    go run main.go
22. open browser and hit localhost:8080/add/35/45
    or localhost:8080/mult/35/45


