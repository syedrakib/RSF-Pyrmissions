# RSF-Pyrmissions

Ridiculously Straight Forward Permissioning system (aka Access Control) for python projects

**Install the package from PyPI** 

    pip install rsf-pyrmissions

**Import the package and create an object of `PermissionsConfiguration` class. We will call it `pc` for this documentation.**
    
    from rsf_pyrmissions.utils import PermissionsConfiguration
    pc = PermissionsConfiguration(is_registration_required=False)

In a production system, we should pass in a boolean `True` for the `is_registration_required` argument in this object instantiation statement. However, for this demo purpose, we will set it to `False` and, hence, avoid any requirements for registrations - we will discuss the importance of registrations a little later below.

---

# Setting privileges for roles and users

We will use random roles and users to assign privileges to them. Note that, there is nothing significant about what / how you call each role / user. It doesn't matter whether a role is called "superuser" or an "intern". The only thing that matters is which arbitrary string denoting a role has been assigned what kind of privileges for an arbitrary string of action. 

    # all managers can remove items
    a_role = "manager"
    an_action = "remove"
    is_allowed = True
    pc.assign_privilege_for_a_role(a_role, an_action, is_allowed)
    
    # no users can remove items
    a_role = "users"
    an_action = "remove"
    is_allowed = False
    pc.assign_privilege_for_a_role(a_role, an_action, is_allowed)
    
    # Alice can always remove items whether she is a manager or a user
    a_user = "alice"
    an_action = "remove"
    is_allowed = True
    pc.assign_privilege_for_a_user(a_user, an_action, is_allowed)
    
It's all arbitrary - you're free to call your roles, users, actions whatever you want. All these privilege assignments and rules have been saved inside the `pc` object. 

# Determining privileges for a role or a user
    
    a_role = "user"
    a_user = "alice"
    an_action = "remove"
    pc.is_allowed(a_role, a_user, an_action) # True
    
    a_role = None # maybe Alice was never even assigned any role in our project
    a_user = "alice"
    an_action = "remove"
    pc.is_allowed(a_role, a_user, an_action) # True, 
    # returns True because alice has been explicitly assigned a privilege for this action

Let's take Bob for example. He is someone who hasn't been assigned any privileges yet.

	a_role = "user" # or could be None if we don't know his role in the project
	a_user = "bob"
	an_action = "remove"
	pc.is_allowed(a_role, a_user, an_action) # False
    
Bob as a user cannot remove items. But managers can. So, if Bob is promoted to become a manager
    
    a_role = "manager"
    a_user = "bob"
    an_action = "remove"
    pc.is_allowed(a_role, a_user, an_action) # True
    
Each of these fields (`a_role`, `a_user`, `an_action`) are compulsory for determining a privilege with `is_allowed()`. However, it is completely normal to leave one or more of these fields as `None` if its value is not known particularly. 

By default, if any role-user-action combination is used which was not previously assigned any privileges, then `False` will be returned
    
    a_role = "supermutant" # supermutants can destroy a universe
    a_user = "mastermutant" # mastermutant is the master of all super mutants
    an_action = "squash_an_apple" 
    pc.is_allowed(a_role, a_user, an_action) # False
    # returns False because this role/user was never assigned any privileges on the first place

# Removing assigned privileges from roles and users

As noted above, let's say managers are currently allowed to remove items.
    
    a_role = "manager"
    a_user = "bob"
    an_action = "remove"
    pc.is_allowed(a_role, a_user, an_action) # True

To unassign this privilege from the role of managers, we can do the following:

    pc.unassign_privilege_for_a_role(a_role, an_action)

Now, if we query `is_allowed` for this action by the same role and same user, we will get `False`.

    pc.is_allowed(a_role, a_user, an_action) # False

Similary, we can also unassign a privilege from a particular user. As noted above, let's say Alice is currently allowed to remove items regardless of her role.
    
    a_role = None # maybe Alice was never even assigned any role in our project
    a_user = "alice"
    an_action = "remove"
    pc.is_allowed(a_role, a_user, an_action) # True

Now, if we unassign this special privilege from Alice and then query for `is_allowed` with the same role and the same user once again, we will get `False` in return.

    pc.unassign_privilege_for_a_user(a_user, an_action)
    pc.is_allowed(a_role, a_user, an_action) # False

Thus, we can **assign** privileges to roles and users. And also **unassign** privileges from roles and users.

***Caution with unassign!!!*** The `unassign_privilege` methods will throw a `LookupError` if the mentioned combination of action with a user/role does not exist in the permissions configuration during the time of callling the `unassign_privilege` methods. The error needs to be caught and handled/dismissed as reasonable.


---

# Fine-grain control with special conditions

So far, Alice was able to remove any item she wanted regardless of whether she was a user or a manager. However, we've just unassigned that privilege from Alice in our last example - sorry Alice.

Now, Bob, as a user, cannot remove any item. But if Bob queries his privilege as a Manager then he can remove any item because the Manager role is allowed to remove items. 

Let's say we want to allow Bob to remove items (whether he is a manager or a user) but only those items which belong to him - not others items. We can call this a condition called `"self_items"`. 

Now, while assigning a privilege to a user or a role, instead of supplying a boolean True/False, we can also supply a string to represent a condition.

    a_user = "bob"
    an_action = "remove"
    required_condition = "self_items"
    pc.assign_privilege_for_a_user(a_user, an_action, required_condition)

**Notice that**, previously we had been submitting a `is_allowed` boolean as the 3rd argument for assigning privileges. Now, we are submitting a `required_condition` string as the 3rd argument for assigning privileges. Both are permissible. 

When queried with `is_allowed()`, you will get either a True or a False in response. However, when fine-grain conditions are involved, it is recommended to query using the `is_allowed_or_required_condition()` method instead - it will return either a `True` or a `False` or the supplied condition that was assigned.

    a_role = "intern"
    a_user = "bob"
    an_action = "remove"
    pc.is_allowed_or_required_condition(a_role, a_user, an_action) # 'self_items'

Any user or any role can be permitted some actions based on some conditions as we saw above. If conditions have been set for the role (or for the user), then the `is_allowed_or_required_condition()` method will return the condition instead of returning a `True` or a `False`.

# Which method to use for determining privileges

Using the `is_allowed()` is recommended for simple use cases where no complex conditions are involved. You will get a response in either `True` or `False`.

Using the `is_allowed_or_required_condition()` is recommended for more robust use cases where complex conditions may be involved. It may return boolean `True`/`False`, or may return a string representing a required condition.

---

# Requiring Registrations

Earlier in the docs above, we had skipped this topic and said it will be covered later. It is possible (and recommended) to enforce some constraints on the kinds of strings that can be used for assigining the roles, users, actions, conditions etc. Since these are just arbitrary strings, while assigning a privilege with a mistyped string parameter, it should raise an error and warn you instead of silently accepting it.

In order to prevent such situations, you can (rather, **you should**) turn on the `is_registration_required` option during object creation and register every expected string in one initialization space before actually using them or querying them later on.

    pc = PermissionsConfiguration(is_registration_required=True)
    # or simply
    pc = PermissionsConfiguration(True)
    # or even simpler
    pc = PermissionsConfiguration() # it defaults to True

Or if you want to change this behavior later on

    pc.is_registration_required(True)
    # or 
    pc.is_registration_required(False)

You can query the value of its current state by simply passing no arguments at all

	pc.is_registration_required()

Register the strings that you expect will be used / needed in your project in the future

    pc.register_roles("manager", "user", "intern")
    pc.register_users("alice", "bob", "carl", "david")
    pc.register_actions("add", "remove", "edit")
    pc.register_conditions("self_items", "all_items")

Hence, later on if you try to assign a privilege with a mistyped parameter (or with a new parameter which has not yet been registered) it will raise an error. 

    pc.assign_privilege_for_a_role("superintern", "remove", True)
	# LookupError: Role 'superintern' is not yet registered in this configuration

    pc.assign_privilege_for_a_user("alice", "remov", True)
	# LookupError: Action 'remov' is not yet registered in this configuration

This helps in various cases where accounting / planning for newly introduced roles, users, actions (before using them) is important. **It is recommended to enforce this requirement in production applications.**

---

# Dumping & Loading (export/import)

**Dumping**

    dump_string = pc.dumps() # produces a json dump of the current config state of the object

This string represents the current state of the `pc` object and it can be easily stored in a regular database or a file system etc to be loaded from later on.

**Loading**

	pc2 = PermissionsConfiguration() # fresh new object 
    pc2.loads(dump_string)
	
This `pc2` object is now an exact replica of the original `pc` object that we first created. We can resume all assigning and querying operations on this new `pc2` object just like we were doing for the original `pc` object.

***CAUTION with Loading!!!*** The `loads()` method will completely replace whatever config state was present in the config object before the `loads()` method was called. So use this method with caution.

---

# Summary

- Privileges can be assigned (and also be unassigned) to/from both users and roles
- Any privilege that has been defined explicitly for a specific user, will override whatever role he/she is a part of - role of the user does not matter if an explicit privilege has been assigned to that user for an action
- A user (or a role) can be completely allowed/disallowed some actions or be allowed on certain conditions only
- Whenever a combination of role/user/action is used which is particularly unknown to the PermissionsConfiguration object, it will return `False` when queried with `is_allowed()` or with `is_allowed_or_required_condition()`.
- There are no automatic heirarchies enforced in any way (such as manager can do whatever a user can, a user can do whatever an intern can etc). You are expected to set the privileges for every action for every role/user explicitly.

# Important note

RSF-Pyrmissions does not keep record of which user has which role, which role has higher privileges than other roles etc. When you use this simple library in your python project, it is expected that your project will keep the important records of which user has which role and who is above whom and what not. 

Your project will determine which user is trying to perform what action, whether the user is fulfiling a condition or not etc. RSF-Pyrmissions will only let you know if `a_user` with `a_role` trying to perform `an_action` is allowed (or disallowed) to perform that action with (or without) a `condition` - simple, no strings attached, ridiculously straight forward.
    
    
    
    
    
    