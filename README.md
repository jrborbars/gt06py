# GT06 Client Features

## Revision History

| Version | Date | Author | Change Description |
| :--- | --- | --- | --- |
| 1.0.0 | 2022-07-12 | Sun Jian | Initial Version |

## GT06 Introduction

### Functional Overview

This project implements the **client** functionality of the GT06 protocol. Users can directly use this feature to perform standard data interaction with the corresponding server.
### Main Features

- Server connection
- Terminal login
- Terminal GPS & LBS positioning information reporting
- Terminal device status information reporting
- Server instruction issuance and response

### Remarks

- The GT06 protocol specifies relatively few messages. This project only implements basic message functions. Custom messages from different service platforms require secondary development, and extended messages also require interface adjustments.
- QuecPython has not implemented Unicode name escaping, while GT06 server downlink messages contain Unicode encoded data. Therefore, escaping cannot be performed; please take note of this.

# TODO
- Make server works
    - translate to async + uvloop
- Testss
