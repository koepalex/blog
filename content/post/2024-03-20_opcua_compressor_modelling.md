---
title: "OPC UA Data Modelling"
date: "2024-03-20"
categories: 
  - "OPC UA"
tags: 
  - "opcua"
  - "siome"
---

In this multi-step tutorial I'd like to explain (i) how to model a machine in OPC UA, (ii) to create an OPC UA server that simulates values for that machinery, (iii) to read data from the server and sending it upstream (e.g. into an cloud) as well as (iv) to detect anomalies.
 
## What is OPC UA?
To answer this question, I like to cite Stefan Hoppe the President and Executive Director of the OPC Foundation:
> OPC Unified Architecture (OPC UA) is the information exchange standard for secure, reliable, manufacturer- and platform-independent industrial communications. It enables data exchange between products from different manufacturers and across operating systems. The OPC UA standard is based on specifications that were developed in close cooperation between manufacturers, users, research institutes and consortia, in order to enable consistent information exchange in heterogeneous systems.
>
>For nearly three decades, OPC has been, and continues to be, the go to connectivity standard in indus-
try. With the advent of the Internet of Things (IoT) era, OPC adoption has also shown growth in new, non-industrial markets. By introducing a Service-Oriented-Architecture (SOA) in industrial automation systems in 2007, OPC UA started to offer a scalable, platform-independent solution for interoperability which combines the benefits of web services and integrated security with a consistent data model.

OPC UA is an IEC standard and is therefore ideally suited for collaboration with other organizations.
The biggest value from my point of view is that OPC UA were able to create an ecosystem for connectivity, it is not just one protocol (opc.tcp) rather can work with different procotols (opc.http, mqtt, ...) and different payload formats (UADP, Json, XML, ...) and even more important today are the so called OPC UA Companion Specs, which provide the best available industrial information model to interact with all sorts for machineries.

## Isn't OPC UA too complex for simple use cases?
Well I believe it is not too complex but let's see a concrete example, we will use an compressor as example for an machine. The compressor is defined by 4 properties
* EnergyConsumption (double)
* ErrorState (boolean) 
* MachineIdentifier (string)
* OilPressure (double)
  
The defacto standard to model something in OPC UA are NodeSet2 XML files, and there are a variaty of tools to generate them. We are going to use the free [Siemens OPC UA Modeling Editor (SiOME)]([https://support.industry.siemens.com/cs/document/109755133/siemens-opc-ua-modeling-editor-(siome)?dti=0&dl=en&lc=de-DE](https://support.industry.siemens.com/cs/document/109755133/siemens-opc-ua-modeling-editor-(siome)?dti=0&dl=en&lc=de-DE)). SiOME is an Windows application that can be downloaded for free after registration and don't need an installation.
 
Start by adding a new namespace, I will use `alexander-koepke.de/opcua/compressor` 
![](/images/siome-new-namespace.png)
![](/images/siome-namespace-dialog.png)
We will add a type called `MyCompressorType`, to do this right click on  _Root/Types/ObjectTypes/BaseObjectType_ and select _Add New ObjectType:_
 
![](/images/siome-new-objecttype-dialog.png)
The new type is automatically selected in the tree and we can start adding fields to it. We will start with the _EnergyConsumption_ by right click on _Root/Types/ObjectTypes/BaseObjectType/MyCompressorType_ and select _Add Child:_
![](/images/siome-add-child-energyconsumption.png)
We changed the _NodeClass_ to **Variable** (other possibilities are **Object** or **Method**) and the _DataType_ to **Double**. OPC UA has a rich type system, that can also be extended by own types, this is one of the reasons why OPC UA is so valuable to industrial communication. Now we finish our type by adding the other properties of the compressor:
![](/images/siome-add-child-errorstate.png)
![](/images/siome-add-child-machineidentifier.png)
![](/images/siome-add-child-oilpressure.png)
... and we are **done** with the modelling and can save the result e.g. as [`MyCompressorType.NodeSet2.xml`](/files/MyCompressorType.NodeSet2.xml).  That was not so complex right? In the next article I will show how to use the NodeSet2 XML file to create an OPC UA server that simulate values for the compressor.

## What would be the difference in "bigger production" environments?
OPC UA provides a big set of existing information models all available under [https://reference.opcfoundation.org](https://reference.opcfoundation.org) (Documents with number bigger or equal to 10000-100  are typically information models). So it is a good starting point look if there is already a companion specification available for the type of "thing" to model and read it. If there is no specific companion spec, it is good practice to use at least **10000-100 Devices** to begin with. The Companion Spec explains the types, interfaces, events, datatypes etc. that are predefined and how to use them.  
Once you found the right companion specification, you could download the related NodeSet2 XML file (and all the NodeSet2 XMLs of companion specifications it references) from [https://github.com/OPCFoundation/UA-Nodeset](https://github.com/OPCFoundation/UA-Nodeset). As example this is the companion specification for Devices (Device Integration): [https://raw.githubusercontent.com/OPCFoundation/UA-Nodeset/latest/DI/Opc.Ua.Di.NodeSet2.xml](https://raw.githubusercontent.com/OPCFoundation/UA-Nodeset/latest/DI/Opc.Ua.Di.NodeSet2.xml)
So the difference would be that you before start creating your own types, you would import all NodeSet2 XMLs of the Companion Specs yoou like to use and reuse the definitions already provided to you. In case that the compressor should be modelled with Device Integration, we would add objects of `IVendorNameplateType` and `ITagNameplateType` as children of the `MyCompressorType`.
