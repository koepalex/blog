---
title: "OPC UA Data Modelling"
date: "2024-03-20"
categories: 
  - "OPC UA"
tags: 
  - "opcua"
  - "siome"
---

In this comprehensive tutorial, we will explore the process of OPC UA data modelling. The tutorial will cover modeling a machine, creating an OPC UA server to simulate machinery values, reading data from the server and transmitting it downstream (e.g., to the cloud), and detecting anomalies.
 
## What is OPC UA?
To answer this question, I like to cite Stefan Hoppe the President and Executive Director of the OPC Foundation:
> OPC Unified Architecture (OPC UA) is the information exchange standard for secure, reliable, manufacturer- and platform-independent industrial communications. It enables data exchange between products from different manufacturers and across operating systems. The OPC UA standard is based on specifications that were developed in close cooperation between manufacturers, users, research institutes and consortia, in order to enable consistent information exchange in heterogeneous systems.
>  
>For nearly three decades, OPC has been, and continues to be, the go to connectivity standard in indus-
try. With the advent of the Internet of Things (IoT) era, OPC adoption has also shown growth in new, non-industrial markets. By introducing a Service-Oriented-Architecture (SOA) in industrial automation systems in 2007, OPC UA started to offer a scalable, platform-independent solution for interoperability which combines the benefits of web services and integrated security with a consistent data model.

OPC UA is an IEC standard and is therefore ideally suited for collaboration with other organizations.
The biggest value from my point of view is that OPC UA were able to create an ecosystem for connectivity, it is not just one protocol (opc.tcp) rather can work with different procotols (opc.http, mqtt, ...) and different payload encodings (UADP, Json, XML, ...) and even more important today are the so called OPC UA Companion Specs, which provide the best available industrial information model to interact with all sorts for machineries.

## Demystifying OPC UA Data Modelling
### Is OPC UA Too Complex?
Despite its comprehensive capabilities, OPC UA is not overly complex, as demonstrated through a concrete example. Consider modeling a compressor with four properties:

* EnergyConsumption (double)
* ErrorState (boolean)
* MachineIdentifier (string)
* OilPressure (double)

The defacto standard to model something in OPC UA are NodeSet2 XML files, and there are a variaty of tools to generate them. We are going to use the free [Siemens OPC UA Modeling Editor (SiOME)]([https://support.industry.siemens.com/cs/document/109755133/siemens-opc-ua-modeling-editor-(siome)?dti=0&dl=en&lc=de-DE](https://support.industry.siemens.com/cs/document/109755133/siemens-opc-ua-modeling-editor-(siome)?dti=0&dl=en&lc=de-DE)). SiOME is an Windows application that can be downloaded for free after registration and don't need an installation. Let's walk through the steps:
 
1. Adding a Namespace

Start by adding a new namespace, e.g., alexander-koepke.de/opcua/compressor.
![](/images/siome-new-namespace.png)
![](/images/siome-namespace-dialog.png)

2. Creating Object Types

Define a new type, such as `MyCompressorType`(right click on  _Root/Types/ObjectTypes/BaseObjectType_ and select _Add New ObjectType_)
 
![](/images/siome-new-objecttype-dialog.png)

3. Configuring Properties

Add the properties and configure each property with the appropriate data type (right click on _Root/Types/ObjectTypes/BaseObjectType/MyCompressorType_ and select _Add Child_).
![](/images/siome-add-child-energyconsumption.png)
Change the _NodeClass_ to **Variable** (other possibilities are **Object** or **Method**) and the _DataType_ to **Double**. OPC UA has a rich type system, that can also be extended by own types, this is one of the reasons why OPC UA is so valuable to industrial communication. Now we finish our type by adding the other properties of the compressor:
![](/images/siome-add-child-errorstate.png)
![](/images/siome-add-child-machineidentifier.png)
![](/images/siome-add-child-oilpressure.png)

4. Saving the Model

Save the model as a NodeSet2 XML file, e.g.,  [`MyCompressorType.NodeSet2.xml`](/files/MyCompressorType.NodeSet2.xml).

By following these steps, users can efficiently model machinery in OPC UA without unnecessary complexity.

## Scaling Up for Production Environments
In larger production environments, OPC UA provides a vast array of existing information models, available at https://reference.opcfoundation.org. These models, typically represented by documents with numbers equal to or greater than **10000-100**, serve as companion specifications, defining predefined types, interfaces, events, and data types.
  
For more sophisticated applications, it's advisable to leverage existing companion specifications before creating custom types. These specifications streamline the modeling process by providing pre-defined structures and definitions. Users can import NodeSet2 XML files from the repository https://github.com/OPCFoundation/UA-Nodeset to incorporate companion specifications into their projects seamlessly.

As example to start the integration of the companion specification for Devices (Device Integration): [https://raw.githubusercontent.com/OPCFoundation/UA-Nodeset/latest/DI/Opc.Ua.Di.NodeSet2.xml](https://raw.githubusercontent.com/OPCFoundation/UA-Nodeset/latest/DI/Opc.Ua.Di.NodeSet2.xml) add instances of `IVendorNameplateType` and `ITagNameplateType` to the `MyCompressorType`.

In conclusion, while OPC UA offers robust capabilities for complex industrial scenarios, tools like SiOME and existing companion specifications simplify the data modeling process, making OPC UA accessible even for simpler use cases.

Stay tuned for the next article, where we'll demonstrate how to use the NodeSet2 XML file to create an OPC UA server that simulates values for the compressor.