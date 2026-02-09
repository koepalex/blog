---
title:       "OPC UA nodeset export"
date:        2026-02-09
tags:        ["opcua", "nodeset2", "nodeset export"]
categories:  ["OPC UA"]
---

# Exporting OPC UA Address Spaces the Easy Way
## Introducing the OPC UA Nodeset2.xml Exporter
Working with OPC UA servers often means dealing with large and complex address spaces. Whether you're testing, simulating, validating companion specifications, or setting up environments for software vendors, having a clean and accurate export of the server’s address space is essential. That’s exactly why the OPC UA Nodeset2.xml Exporter was created.
This lightweight tool (available on [GitHub koepalex/OpcUaNodesetExporter](https://github.com/koepalex/OpcUaNodesetExporter)) enables you to export the full OPC UA address space of a running server into the standardized NodeSet2 XML format — the same format used by OPC Foundation companion specifications.
In this post, we’ll walk through the motivation behind the tool, what it does, and why it may become a valuable addition to your OPC UA toolkit.

## Why Another Address Space Exporter?
OPC UA servers expose rich information models. But when you're working in engineering, simulation, or testing workflows, you often need that information model in a portable, machine-readable form. Existing SDKs and tools don't always make this simple or require extra license. The tool was created to solve several real-world challenges:

### ✔️ Simulation & Testing
When building simulation servers, it’s often necessary to replicate the address space of an existing system, either fully or partially. A NodeSet2 export allows simulation servers to reconstruct the same nodes, references, and metadata — enabling more realistic behavior.

### ✔️ Verifying Companion Specification Implementations
Many devices claim to implement certain OPC UA Companion Specifications (Euromap, PackML, Machine Tools, Weihenstephan Standards, etc.). With a standardized NodeSet2 export You can (i) verify whether the server actually provides all required nodes, (ii) Tools can validate the address space against the specification (iii) Manual troubleshooting becomes much simpler.

### ✔️ Troubleshooting & Diagnostics
When something in your OPC UA model behaves unexpectedly, having a NodeSet2 export enables Structured offline analysis, Comparison with known-good models, Sharing the exact address space with colleagues, vendors, or the community. Instead of “it works on my machine,” you can provide the actual exported model.

### ✔️ Providing Input for Software Vendors
Many vendors building OPC UA-based tools — data ingestion pipelines, digital twins, MES connectors, modeling software — accept NodeSet2 files as input. By exporting your server 

## What the Tool Does
The OPC UA Nodeset Exporter connects to an OPC UA server and extracts:

* All nodes in the address space
* Full type definitions
* References and relationships
* Metadata such as NodeIds, BrowseNames, DisplayNames
* DataTypes, Objects, Variables, Methods, and Views
* Namespace mappings

The final output is a conformant NodeSet2.xml document, which can be used in OPC UA SDKs, Modeling tools, Simulation frameworks, Validation tools, as well as Automated test systems. While the OPC Foundation and some SDKs provide tools to generate NodeSet2 from *ModelDesign files*, exporting directly from a **running server** is a different use case — and that’s where this tool fills the gap.

If you work with OPC UA in engineering, automation, IIoT, or manufacturing IT, this exporter can help streamline your workflow.

## Getting Started
You’ll find the tool here:
👉 [https://www.nuget.org/packages/OpcUaNodesetExporter/](https://www.nuget.org/packages/OpcUaNodesetExporter/)

