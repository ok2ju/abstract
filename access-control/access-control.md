# Access control

## Types of access control

- [MAC](#mac) - Mandatory Access Control;
- [RBAC](#rbac) - Role-Based Access Control;
- [ABAC](#abac) - Attribute-based access control;
- [DAC](#dac) - Discretionary Access Control;
- [ACL](#acl) - Access control list;

## MAC

MAC stands for Mandatory Access Control (MAC). its a security model where users are given permissions to resources by an admin or root. These permissions can ONLY be granted by the root user or administrator.Only an administrator can grant permissions or right to objects and resources. In this model only the administrator can change the object user security clearance or security label

## RBAC

RBAC in the Role-Based Access Control (RBAC) model, access to resources is based on the role assigned to a user. In this model, an administrator assigns a user to a role that has certain predetermined right and privileges. Because of the user's association with the role, the user can access certain resources and perform specific tasks. RBAC is also known as Non-Discretionary Access Control. The roles assigned to users are centrally administered.

## ABAC

Attribute-based access control is an alternative framework, that evolved out of RBAC. Rather than creating specific roles, ABAC involves setting conditions to automatically grant permissions, based on users’ existing attributes.
These could be variables like their job title, location, or department. It could even be a changing attribute, like the current device or network that a user access your platform from.

ABAC is also sometimes referred to as dynamic access control.

This is because a user’s permissions automatically change when one of the tied attributes changes. So for example, an employee might automatically gain increased permissions, if they get a promotion. At least, if their permissions are tied to their job title.

## DAC

DAC In the Discretionary Access Control (DAC) model, access to resources is based on user's identity. A user is granted permissions to a resource by being placed on an access control list (ACL) associated with resource. An entry on a resource's ACL is known as an Access Control Entry (ACE). When a user (or group) is the owner of an object in the DAC model, the user can grant permission to other users and groups. The DAC model is based on resource ownership.

## ACL

Access control lists are a very simplistic access control system. Today, ACL systems are less common for applications but are still used in networks and legacy file systems.

Essentially, this is a table that lists the permissions attached to different objects and resources.

So, an ACL lists three things:

- Controlled objects;
- The users that can access them;
- The actions these users can carry out - ie read, update, create, or delete;
- Each user must have their own entry. This is then linked to the security attributes of all relevant objects.

## Sources

- [Role-Based Access Control](https://budibase.com/blog/app-building/role-based-access-control)
