---
title:       "Validate OPC UA Address Spaces Before Integration"
date:        2026-07-17
tags:        ["opcua", "address space", "validation", "companion specifications"]
categories:  ["OPC UA"]
---

# Validate OPC UA Address Spaces Before Integration

An OPC UA server can be reachable, browsable, and able to return values while its
address space is still modeled incorrectly. A mandatory component may be missing,
a Variable may use the wrong DataType, or a node may appear at the wrong place in
the hierarchy. These problems are easy to overlook in a client browser, but they
often surface later when another application relies on the information model
instead of only reading individual NodeIds.

[OpcUaAddressSpaceChecker](https://www.nuget.org/packages/OpcUaAddressSpaceChecker/)
is a .NET global tool that detects these structural problems directly on a running
OPC UA server.

## Why Address Space Validation Matters

OPC UA information models are contracts. A TypeDefinition describes which children
an instance must or may expose, their BrowseNames, NodeClasses, DataTypes, references,
and ModellingRules. Companion specifications build on these contracts so that clients
can understand a device without vendor-specific configuration.

A server implementation may violate that contract even though basic smoke tests pass.
Typical examples include:

* a mandatory child declared by a type is absent;
* an Object or Variable has no TypeDefinition;
* a Property is linked with `HasComponent` instead of `HasProperty`;
* a Variable has an incompatible DataType or ValueRank;
* a nested declaration is exposed directly below the instance instead of at its
  defined BrowsePath;
* an instance uses an abstract type;
* companion-specification entry points or grouping objects are placed incorrectly.

Such issues reduce interoperability. Generic clients, code generators, analytics
pipelines, and digital-twin integrations cannot safely infer the intended semantics.
Finding the problem during server development is much cheaper than debugging it in
an integration environment.

## How the Checker Works

The checker connects to the endpoint, browses the address space, and compares Object
and Variable instances with their type declarations. By default, it reads the type
model from the server's standard Types folder. This makes the first validation run
simple and also checks the model actually exposed by the server.

For servers that do not expose enough type information, one or more NodeSet2 files
can be supplied as an explicit type-model override. This is useful when validating
against a companion specification or a model used during development.

The validation includes generic OPC UA modeling rules and selected checks for
companion specifications such as Device Integration, Machinery, and Pumps. The
complete and current rule catalog is maintained in the
[project README](https://github.com/koepalex/OpcUaAddressSpaceChecker).

## Install and Run

Install the tool from NuGet:

```powershell
dotnet tool install --global OpcUaAddressSpaceChecker
```

Then validate an anonymously accessible endpoint:

```powershell
opcua-check-address-space --endpoint opc.tcp://localhost:4840
```

The console report is useful while developing a server because each finding identifies
the affected node, violated rule, severity, and expected model structure.

To validate against explicit NodeSet2 models, pass the main model and a directory
containing its dependencies:

```powershell
opcua-check-address-space `
  --endpoint opc.tcp://localhost:4840 `
  --nodeset .\models\MyModel.NodeSet2.xml `
  --nodeset-dir .\models\dependencies
```

The tool also supports username and certificate authentication as well as signed and
encrypted endpoints. Credentials can be provided through environment variables or
standard input so they do not have to appear in shell history or build logs.

## Use It in Continuous Integration

Address space validation is especially valuable as an automated integration test.
Start the server or simulator, wait until its OPC UA endpoint is available, and run
the checker before publishing or deploying the server.

For example, create a SARIF report and fail the job when warnings or errors are found:

```powershell
opcua-check-address-space `
  --endpoint $env:OPCUA_ENDPOINT `
  --output-format sarif `
  --output address-space-results.sarif `
  --severity-threshold warning
```

SARIF can be consumed by code-scanning tools, while JSON is convenient for custom
automation. A Markdown report works well as a human-readable build artifact because
it groups findings by namespace and includes links to the relevant OPC Foundation
references.

The process exit code distinguishes validation findings from connection, model-loading,
and command-line errors. This allows a pipeline to fail for the right reason instead
of treating every unsuccessful run as the same problem.

## A Practical Validation Workflow

Run the checker early against a development server and resolve errors that violate the
type contract. Review warnings separately: an unexpected child might be a legitimate
vendor extension, but it might also reveal a misplaced node or an incomplete type
definition.

When a deviation is intentional, exclude only the relevant rule rather than disabling
validation altogether. Keeping the remaining checks active preserves protection
against regressions as the address space evolves.

For companion-specification implementations, combine the checker with a NodeSet2 export
of the running server. The checker provides immediate, actionable findings, while the
export remains useful for offline inspection, comparison, and sharing a reproducible
model with other teams.

## Conclusion

OPC UA interoperability depends on more than establishing a secure session and reading
values. The address space itself must honor the contracts expressed by its types and
companion specifications.

OpcUaAddressSpaceChecker turns those contracts into repeatable tests that can run on a
developer machine or in CI. It helps find modeling defects where they originate:
before clients, data pipelines, or customers have to compensate for them.

The tool is available on
[NuGet](https://www.nuget.org/packages/OpcUaAddressSpaceChecker/), and its source code,
full usage reference, and rule catalog are available on
[GitHub](https://github.com/koepalex/OpcUaAddressSpaceChecker).
