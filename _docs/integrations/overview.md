---
title: Overview
category: Nexd Modules
order: 1
---

Nexd Modules provide the ability for members of the extended Nexd team to provide functionality to the core product.  We will eventually allow Node Modules to be written in a number of programming languages.  Currently only Node is fully supported and Python supports only signal generation.

Each module contains code, and configuration to control how product, gets and stores raw data from external system, dynamically calculates interesting properties, and signals about the data, and how to display the dynamic information.

The code in the integration portion of a Nexd Module often connects to an external system and sends the content from the external system to Nexd, along with the activity for the content in the form of content events.  Code can also be written to generate complicated signals.

Nexd Module’s also define content types generated by the integration, boards to display the content types along with definition of dynamic properties to be generated for the board, and signals to be generated based on the content, and board’s dynamic context.

Nexd Modules can be developed by anyone on sandbox Nexd team accounts (non-production).  Production team accounts require approved Nexd Modules whose code and configuration is committed to a Nexd controlled GitHub account and will be run and managed by Nexd.  
