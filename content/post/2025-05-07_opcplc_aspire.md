---
title:       "Running OPC UA server simulation in dotnet aspire"
date:        2025-05-07
tags:        ["OPCPLC", "aspire"]
categories:  ["OPC UA", "dotnet" ]
---

# Simulating an OPC UA Server with .NET Aspire and OPC PLC

Deploying an OPC UA server simulation is a common need during development and testing of industrial IoT applications. 
Recently, a customer asked how to set up such a simulation using **.NET Aspire**, in order to streamline development 
workflows and easily monitor system components, logs, metrics, and inter-service communication.

.NET Aspire provides an ideal environment for orchestrating microservices and dependencies, making it a great fit for 
hosting a simulated OPC UA server. For the server simulation, I use the free and open-source 
[OPC PLC](https://github.com/Azure-Samples/iot-edge-opc-plc) provided by Microsoft. While it's possible to run the 
server from source, I prefer using the containerized version published on the Microsoft Container Registry (MCR), 
which integrates more easily into an Aspire-based solution.

---

## Integrating OPC PLC with .NET Aspire

In a .NET Aspire AppHost project, we can add the OPC PLC container as a reference, define its endpoint, and configure 
all necessary startup parameters through container arguments.

Here’s how to set it up:

```csharp
var port = 50000;
var numberOfSlowNodes = 50;
var numberOfFastNodes = 50;
var aspireOtelEndpoint = "https://localhost:21222";

var opcplc = builder
    .AddContainer("opcplc", "mcr.microsoft.com/iotedge/opc-plc", "2.12.32")
    .WithEndpoint(port: port, targetPort: port, scheme: "opc.tcp", name: "default")
    .WithArgs("--ph=opcplc")                           // Hostname for OPC PLC
    .WithArgs("--cdn=opcplc")                          // Additional hostnames or IPs for the certificate
    .WithArgs("--autoaccept")                          // Auto-accept all certificates
    .WithArgs($"--sn={numberOfSlowNodes}")             // Number of slow-changing nodes
    .WithArgs("--sr=10")                               // Slow node update rate (10s)
    .WithArgs($"--fn={numberOfFastNodes}")             // Number of fast-changing nodes
    .WithArgs("--veryfastrate=1000")                   // Fast node update rate (1s)
    .WithArgs("--gn=5")                                // Number of deterministic GUID nodes
    .WithArgs($"--pn={port}")                          // Server port
    .WithArgs("--maxsessioncount=100")                 // Max session count
    .WithArgs("--maxsubscriptioncount=100")            // Max subscriptions
    .WithArgs("--maxqueuedrequestcount=2000")          // Max queued requests
    .WithArgs("--ses")                                 // Simulate simple events
    .WithArgs("--alm")                                 // Simulate alarms
    .WithArgs("--at=FlatDirectory")                    // Use flat directory for certs
    .WithArgs("--drurs")                               // Don't reject unknown revocation status
    .WithArgs($"--otlpee={aspireOtelEndpoint}")        // OpenTelemetry endpoint for Aspire
    .WithArgs("--otlpei=60")                           // OTEL export interval (seconds)
    .WithArgs("--otlpep=grpc");                        // OTEL export protocol
```

---

## Connecting the Client Application

To consume the OPC UA server from another application within the Aspire AppHost, you need to retrieve the endpoint 
and use `.WithReference()` to wire it up:

```csharp
var opcplcEndpoint = opcplc.GetEndpoint("default");

var myApp = builder
    .AddProject<Projects.MyOpcUaClientApplication>("app")
    .WithReference(opcplcEndpoint);
```

Under the hood, `.WithReference()` injects the endpoint into the target application via an environment variable. The 
format is `services__{sourceResourceName}__{endpointName}__{endpointIndex}` Since the `scheme` (e.g., `opc.tcp`) is not 
valid in environment variable names, we provide a friendly name (`"default"`) when defining the endpoint earlier.
In this example, the resulting environment variable will be `services_opcplc_default_0` and it will contain the endpoint 
value `opc.tcp://localhost:50000`. Your client application can then read this variable to dynamically connect to the 
simulated OPC UA server.

---

This approach allows you to simulate a realistic industrial setup with configurable node behavior and integrate 
telemetry and monitoring through OpenTelemetry, all while staying within the developer-friendly .NET Aspire ecosystem.
