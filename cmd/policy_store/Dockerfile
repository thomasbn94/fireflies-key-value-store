FROM golang:alpine AS builder
ENV GOBIN $GOPATH/bin
RUN apk update
RUN apk add git
RUN apk --no-cache add ca-certificates

# Set necessary environmet variables needed for our image
ENV GO111MODULE=auto \
    CGO_ENABLED=0 \
    GOOS=linux \
    GOARCH=amd64

# Move to working directory /build
WORKDIR /build

# Copy and download dependency using go mod
COPY go.mod .
COPY go.sum .
RUN go mod download

# Run test
#RUN go test ./...

# Build the application
#RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .
COPY ./policy.go ./policy.go
COPY ./core/datasetmanager ./core/datasetmanager
COPY ./core/membershipmanager ./core/membershipmanager
COPY ./core/message ./core/message
COPY ./core/netutil ./core/netutil
COPY ./core/comm ./core/comm
COPY ./core/policystore ./core/policystore
COPY ./core/statesync ./core/statesync
COPY ./protobuf ./protobuf
COPY ./core/util ./core/util
COPY ./keyvault.go ./keyvault.go
COPY ./cmd/policy_store/launcher.go ./cmd/policy_store/launcher.go
COPY ./cmd/policy_store/config/lohpi_config.yaml ./cmd/policy_store/config/lohpi_config.yaml
COPY ./cmd/policy_store/config/lohpi_config.dev.yaml ./cmd/policy_store/config/lohpi_config.dev.yaml

RUN go build -o policy_store ./cmd/policy_store/launcher.go

# Move to /dist directory as the place for resulting binary folder
WORKDIR /dist

# Copy binary from build to main folder
RUN cp /build/policy_store .

############################
# STEP 2 build a small image
############################
FROM scratch
COPY --from=builder /dist/policy_store /
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

# Copy the configuration files
COPY ./cmd/policy_store/config/ifrit_config.yaml /var/tmp/ifrit_config.yaml
COPY ./cmd/policy_store/config/lohpi_config.yaml ./config/lohpi_config.yaml
COPY ./cmd/policy_store/config/lohpi_config.dev.yaml ./config/lohpi_config.dev.yaml
COPY ./cmd/policy_store/crypto/lohpi/owncert.pem crypto/lohpi/owncert.pem
COPY ./cmd/policy_store/crypto/lohpi/cacert.pem crypto/lohpi/cacert.pem
COPY ./cmd/policy_store/crypto/lohpi/key.pem crypto/lohpi/key.pem
COPY ./cmd/policy_store/crypto/ifrit/*.pem crypto/ifrit/

# Command to run the executable
ENTRYPOINT ["/policy_store", "-c", "config/lohpi_config.yaml"]


