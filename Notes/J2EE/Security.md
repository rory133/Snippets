| Annotation        |Bean |Method           | Description  |
| ------------- |:---:|:-------------:| :-----|
| @PermitAll      |X| X | Indicates that the given method (or the entire bean) is accessible by everyone (all roles are permitted).|
| @DenyAll      |X| X      |   Indicates that no role is permitted to execute the specified method or all methods of the bean (all roles are denied). This can be useful if you want to deny access to a method in a certain environment
(e.g., the method launchNuclearWar() should only be allowed in production but not in a test environment). |
| @RolesAllowed |X| X      |    Indicates that a list of roles is allowed to execute the given method (or the entire bean).|
| @DeclareRoles |X|-      |  Defines roles for security checking. |
| @RunAs |X| -     |  Temporarily assigns a new role to a principal. |
