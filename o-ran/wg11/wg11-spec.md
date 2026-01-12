# WG11 Official Specifications & Security Requirements (A Study Notes)

> Source: 
> - O-RAN.WG11.TR.Security-Near-RT-RIC-xApps.0-R004-v06.00 (@SPEC1) 
> - O-RAN.WG11.TR.Threat-Modeling.O-R004-v05.00 (@SPEC2)
> - O-RAN.WG11.TR.O-CLOUD-Security-Analysis.0-R004-v07.00 (@SPEC3)
> - O-RAN.WG11.TS.SRCS.0-R004-v13.00 (@SPEC4)

## WG11 Overview
WG11 is the O-RAN Alliance work group responsible for security across the entire O-RAN architecture. **Its main role is to define how security should be applied in a disaggregated, cloud-native, and multi-vendor RAN environment**. WG11 does not design new network functions, but instead provides the security foundation that all other O-RAN work groups must follow. This includes setting security requirements, identifying threats, and defining control mechanisms that ensure interoperability and trust between components. 

### WG11 Mandate And Responsibilities
The objectives of WG11 focus on protecting O-RAN systems end-to-end, from physical infrastructure to software applications. WG11 develops formal security requirements through Technical Specifications, while Technical Reports are used to study threats, risks, and architectural weaknesses. These reports explain why certain security controls are needed and provide context for the requirements defined in the specifications. WG11 also aligns O-RAN security with modern principles such as zero-trust, strong identity management, and cloud-native security practices.

### Security responsibilities in O-RAN
In terms of responsibilities, WG11 covers security for all major O-RAN components, including the Near-RT RIC, Non-RT RIC, SMO, O-Cloud, and the O-CU/O-DU/O-RU functions. It defines how these components should authenticate each other, how data should be protected in transit and at rest, and how access to functions and interfaces should be controlled. Interfaces such as A1, E2, O1, O2, and Open Fronthaul are treated as critical attack surfaces and must follow consistent security rules defined by WG11.

</br>

Overall, WG11 acts as the central authority for security in O-RAN. Its specifications define what is mandatory for secure deployments, while its reports provide the threat analysis and reasoning behind those requirements. Without WG11, the openness and flexibility of O-RAN would significantly increase security risks, especially in environments with multiple vendors and cloud-based components.

## Architecture in Scope
- O-RU, O-DU, O-CU, SMO
- Near-RT RIC, Non-RT RIC

## Interfaces in Scope
- Open Fronthaul
- E2
- A1
- O1
- O2

## Security Principles (Normative)
- Zero Trust
- Mutual authentication
- Encryption
- RBAC
- Secure onboarding
- Continuous monitoring

## Interface Security Requirements
### E2 Interface
- Authentication requirements
- Integrity and confidentiality expectations

### A1 Interface
- Policy security requirements

### O1 / O2 Interfaces
- Management and orchestration security

## Trust & Identity Model
- Trust boundaries
- Identity management expectations
- Certificate / key assumptions

## Cloud-Native Security
- Container isolation
- Platform trust
- Lifecycle security

## AI / ML Security Considerations
- Data integrity requirements
- Model lifecycle considerations
- rApp / xApp trust assumptions

## Normative vs Informative Notes
- MUST / SHALL requirements
- SHOULD / MAY recommendations

## Open Items / Ambiguities (from specs)
- Areas not clearly defined
- Security left to implementation