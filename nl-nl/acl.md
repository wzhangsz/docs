---
layout: default
language: 'nl-nl'
version: '4.0'
upgrade: '#acl'
title: 'Toegangscontrolelijst (ACL)'
---

# Access Control Lists Component

* * *

## Toegangscontrolelijst (ACL)

[Phalcon\Acl](api/Phalcon_Acl) biedt een eenvoudige en lichtgewicht beheer van toegangscontrole en machtigingen. [Toegangscontrolelijsten](https://en.wikipedia.org/wiki/Access_control_list) (ACL) geven een applicatie toegang tot de gebieden en de onderliggende objecten van aanvragen.

Kortom, ACL's heeft twee objecten: Het object dat toegang nodig heeft, en het object we toegang tot willen. In the programming world, these are usually referred to as Roles and Components. In the Phalcon world, we use the terminology [Role](api/Phalcon_Acl_Role) and [Component](api/Phalcon_Acl_Component).

> **Use Case**
> 
> Een boekhoudkundige toepassing moet verschillende groepen gebruikers toegang geven tot verschillende gebieden van de toepassing.
> 
> **Role** - Administrator Access - Accounting Department Access - Manager Access - Guest Access
> 
> **Component** - Login page - Admin page - Invoices page - Reports page
{:.alert .alert-info}

As seen above in the use case, an [Role](api/Phalcon_Acl_Role) is defined as who needs to access a particular [Component](api/Phalcon_Acl_Component) i.e. an area of the application. A [Component](api/Phalcon_Acl_Component) is defined as the area of the application that needs to be accessed.

Using the [Phalcon\Acl](api/Phalcon_Acl) component, we can tie those two together, and strengthen the security of our application, allowing only specific roles to be bound to specific components.

## Creating an ACL

[Phalcon\Acl](api/Phalcon_Acl) uses adapters to store and work with roles and components. De enige beschikbare adapter op dit moment is [Phalcon\Acl\Adapter\Memory](api/Phalcon_Acl_Adapter_Memory). Een memory adapter zorg voor een aanzienlijke verhoging in de snelheid wanneer de ACL wordt benaderd, maar heeft ook nadelen. Het grootste nadeel is dat het geheugen niet persistent is, dus de ontwikkelaar moet een opslagstrategie voor de ACL-gegevens implementeren, zodat de ACL niet op elk verzoek wordt gegenereerd. Dit kan gemakkelijk leiden tot vertragingen en onnodige verwerking, vooral als de ACL vrij groot en/of opgeslagen is in een database of bestand systeem.

Phalcon biedt ook een gemakkelijke manier voor ontwikkelaars om hun eigen adapters te bouwen door de [Phalcon\Acl\AdapterInterface](api/Phalcon_Acl_AdapterInterface) interface te implementeren.

### In actie

De [Phalcon\Acl](api/Phalcon_Acl) constructor neemt als eerste parameter een adapter die wordt gebruikt om de gegevens op te halen die nodig zijn voor de toegangscontrolelijst.

```php
<?php

use Phalcon\Acl\Adapter\Memory as AclList;

$acl = new AclList();
```

Er zijn twee voor zichzelf sprekende acties die de [Phalcon\Acl](api/Phalcon_Acl) heeft: - `Phalcon\Acl::ALLOW` - `Phalcon\Acl::DENY`

The default action is **`Phalcon\Acl::DENY`** for any [Role](api/Phalcon_Acl_Role) or [Component](api/Phalcon_Acl_Component). This is on purpose to ensure that only the developer or application allows access to specific components and not the ACL component itself.

```php
<?php

use Phalcon\Acl;
use Phalcon\Acl\Adapter\Memory as AclList;

$acl = new AclList();

// Default action is deny access

// Change it to allow
$acl->setDefaultAction(
    Acl::ALLOW
);
```

## Adding Roles

As mentioned above, a [Phalcon\Acl\Role](api/Phalcon_Acl_Role) is an object that can or cannot access a set of [Component](api/Phalcon_Acl_Component) in the access list.

There are two ways of adding roles to our list. * by using a [Phalcon\Acl\Role](api/Phalcon_Acl_Role) object or * using a string, representing the name of the role

To see this in action, using the example outlined above, we will add the relevant [Phalcon\Acl\Role](api/Phalcon_Acl_Role) objects in our list:

```php
<?php

use Phalcon\Acl;
use Phalcon\Acl\Adapter\Memory as AclList;
use Phalcon\Acl\Role;

$acl = new AclList();

/**
 * Create some Roles.
 * 
 * The first parameter is the name of the role, 
 * the second is an optional description
 */

$roleAdmins     = new Role('admins', 'Administrator Access');
$roleAccounting = new Role('accounting', 'Accounting Department Access'); 

/**
 * Add these roles in the list 
 */
$acl->addRole($roleAdmins);
$acl->addRole($roleAccounting);

/**
 * Add roles without creating an object first 
 */
$acl->addRole('manager');
$acl->addRole('guest');
```

## Adding Components

A [Component](api/Phalcon_Acl_Component) is the area of the application where access is controlled. In a MVC application, this would be a Controller. Although not mandatory, the [Phalcon\Acl\Component](api/Phalcon_Acl_Component) class can be used to define components in the application. Also it is important to add related actions to a component so that the ACL can understand what it should control.

There are two ways of adding components to our list. * by using a [Phalcon\Acl\Component](api/Phalcon_Acl_Component) object or * using a string, representing the name of the role

Similar to the `addRole`, `addComponent` requires a name for the component and an optional description.

```php
<?php

use Phalcon\Acl;
use Phalcon\Acl\Adapter\Memory as AclList;
use Phalcon\Acl\Component;

$acl = new AclList();

/**
 * Create some Components and add their respective actions in the ACL
 */
$admin   = new Component('admin', 'Administration Pages');
$reports = new Component('reports', 'Reports Pages');

/**
 * Add the components to the ACL and attach them to relevant actions 
 */

$acl->addComponent(
    $admin,
    [
        'dashboard',
        'users',
    ]
);

$acl->addComponent(
    $reports,
    [
        'list',
        'add',
    ]
);

/**
 * Add components without creating an object first 
 */

$acl->addComponent(
    'admin',
    [
        'dashboard',
        'users',
    ]
);

$acl->addComponent(
    'reports',
    [
        'list',
        'add',
    ]
);
```

## Defining Access Controls

After both the `Roles` and `Components` have been defined, we need to tie them together so that the access list can be created. This is the most important step in the role since a small mistake here can allow access to roles for components that the developer does not intend to. As mentioned earlier, the default access action for [Phalcon\Acl](api/Phalcon_Acl) is `Acl::DENY`, following the [whitelist](https://en.wikipedia.org/wiki/Whitelisting) approach.

To tie `Roles` and `Components` together we use the `allow()` and `deny()` methods exposed by the [Phalcon\Acl\Memory](api/Phalcon_Acl_Memory) class.

```php
<?php

use Phalcon\Acl;
use Phalcon\Acl\Adapter\Memory as AclList;
use Phalcon\Acl\Role;
use Phalcon\Acl\Component;

$acl = new AclList();

/**
 * Add the roles
 */
$acl->addRole('manager');
$acl->addRole('accounting');
$acl->addRole('guest');


/**
 * Add the Components
 */

$acl->addComponent(
    'admin',
    [
        'dashboard',
        'users',
        'view',
    ]
);

$acl->addComponent(
    'reports',
    [
        'list',
        'add',
        'view',
    ]
);

$acl->addComponent(
    'session',
    [
        'login',
        'logout',
    ]
);

/**
 * Now tie them all together 
 */
$acl->allow('manager', 'admin', 'users');
$acl->allow('manager', 'reports', ['list', 'add']);
$acl->allow('*', 'session', '*');
$acl->allow('*', '*', 'view');

$acl->deny('guest', '*', 'view');
```

What the above lines tell us:

```php
$acl->allow('manager', 'admin', 'users');
```

For the `manager` role, allow access to the `admin` component and `users` action. To bring this into perspective with a MVC application, the above line says that the group `manager` is allowed to access the `admin` controller and `users` action.

```php
$acl->allow('manager', 'reports', ['list', 'add']);
```

You can also pass an array as the `action` parameter when invoking the `allow()` command. The above means that for the `manager` role, allow access to the `reports` component and `list` and `add` actions. Again to bring this into perspective with a MVC application, the above line says that the group `manager` is allowed to access the `reports` controller and `list` and `add` actions.

```php
$acl->allow('*', 'session', '*');
```

Wildcards can also be used to do mass matching for roles, components or actions. In the above example, we allow every role to access every action in the `session` component. This command will give access to the `manager`, `accounting` and `guest` roles, access to the `session` component and to the `login` and `logout` actions.

```php
$acl->allow('*', '*', 'view');
```

Similarly the above gives access to any role, any component that has the `view` action. In a MVC application, the above is the equivalent of allowing any group to access any controller that exposes a `viewAction`.

> Please be **VERY** careful when using the `*` wildcard. It is very easy to make a mistake and the wildcard, although it seems convenient, it may allow users to access areas of your application that they are not supposed to. The best way to be 100% sure is to write tests specifically to test the permissions and the ACL. These can be done in the `unit` test suite by instantiating the component and then checking the `isAllowed()` if it is `true` or `false`.
> 
> [Codeception](https://codeception.com) is the chosen testing framework for Phalcon and there are plenty of tests in our github repository (`tests` folder) to offer guidance and ideas.
{:.alert .alert-danger}

```php
$acl->deny('guest', '*', 'view');
```

For the `guest` role, we deny access to all components with the `view` action. Despite the fact that the default access level is `Acl::DENY` in our example above, we specifically allowed the `view` action to all roles and components. This includes the `guest` role. We want to allow the `guest` role access only to the `session` component and the `login` and `logout` actions, since `guests` are not logged into our application.

```php
$acl->allow('*', '*', 'view');
```

This gives access to the `view` access to everyone, but we want the `guest` role to be excluded from that so the following line does what we need.

```php
$acl->deny('guest', '*', 'view');
```

## Querying an ACL

Once the list has been defined, we can query it to check if a particular role has access to a particular component and action. To do so, we need to use the `isAllowed()` method.

```php
<?php

use Phalcon\Acl;
use Phalcon\Acl\Adapter\Memory as AclList;
use Phalcon\Acl\Role;
use Phalcon\Acl\Component;

$acl = new AclList();

/**
 * Setup the ACL
 */
$acl->addRole('manager');
$acl->addRole('accounting');
$acl->addRole('guest');

$acl->addComponent(
    'admin',
    [
        'dashboard',
        'users',
        'view',
    ]
);

$acl->addComponent(
    'reports',
    [
        'list',
        'add',
        'view',
    ]
);

$acl->addComponent(
    'session',
    [
        'login',
        'logout',
    ]
);

$acl->allow('manager', 'admin', 'users');
$acl->allow('manager', 'reports', ['list', 'add']);
$acl->allow('*', 'session', '*');
$acl->allow('*', '*', 'view');

$acl->deny('guest', '*', 'view');

// ....



// true - defined explicitly
$acl->isAllowed('manager', 'admin', 'dashboard');

// true - defined with wildcard
$acl->isAllowed('manager', 'session', 'login');

// true - defined with wildcard
$acl->isAllowed('accounting', 'reports', 'view');

// false - defined explicitly
$acl->isAllowed('guest', 'reports', 'view');

// false - default access level
$acl->isAllowed('guest', 'reports', 'add');
```

## Function based access

Depending on the needs of your application, you might need another layer of calculations to allow or deny access to users through the ACL. The method `isAllowed()` accepts a 4th parameter which is a callable such as an anonymous function.

To take advantage of this functionality, you will need to define your function when calling the `allow()` method for the role and component you need. Assume that we need to allow access to all `manager` roles to the `admin` component except if their name is 'Bob' (Poor Bob!). To achieve this we will register an anonymous function that will check this condition.

```php
<?php

use Phalcon\Acl;
use Phalcon\Acl\Adapter\Memory as AclList;
use Phalcon\Acl\Role;
use Phalcon\Acl\Component;

$acl = new AclList();

/**
 * Setup the ACL
 */
$acl->addRole('manager');

$acl->addComponent(
    'admin',
    [
        'dashboard',
        'users',
        'view',
    ]
);

// Set access level for role into components with custom function
$acl->allow(
    'manager',
    'admin',
    'dashboard',
    function ($name) {
        return boolval('Bob' !== $name);
    }
);
```

Now that the callable is defined in the ACL, we will need to call the `isAllowed()` method with an array as the fourth parameter:

```php
<?php

use Phalcon\Acl;
use Phalcon\Acl\Adapter\Memory as AclList;
use Phalcon\Acl\Role;
use Phalcon\Acl\Component;

$acl = new AclList();

/**
 * Setup the ACL
 */
$acl->addRole('manager');

$acl->addComponent(
    'admin',
    [
        'dashboard',
        'users',
        'view',
    ]
);

// Set access level for role into components with custom function
$acl->allow(
    'manager',
    'admin',
    'dashboard',
    function ($name) {
        return boolval('Bob' !== $name);
    }
);

// Returns true
$acl->isAllowed(
    'manager',
    'admin',
    'dashboard',
    [
        'name' => 'John',
    ]
);

// Returns false
$acl->isAllowed(
    'manager',
    'admin',
    'dashboard',
    [
        'name' => 'Bob',
    ]
);
```

> The fourth parameter must be an array. Each array element represents a parameter that your anonymous function accepts. The key of the element is the name of the parameter, while the value is what will be passed as the value of that the parameter of to the function.
{:.alert .alert-info}

You can also omit to pass the fourth parameter to `isAllowed()` if you wish. The default action for a call to `isAllowed()` without the last parameter is `Acl::DENY`. To change this behavior, you can make a call to `setNoArgumentsDefaultAction()`:

```php
<?php

use Phalcon\Acl;
use Phalcon\Acl\Adapter\Memory as AclList;
use Phalcon\Acl\Role;
use Phalcon\Acl\Component;

$acl = new AclList();

/**
 * Setup the ACL
 */
$acl->addRole('manager');

$acl->addComponent(
    'admin',
    [
        'dashboard',
        'users',
        'view',
    ]
);

// Set access level for role into components with custom function
$acl->allow(
    'manager',
    'admin',
    'dashboard',
    function ($name) {
        return boolval('Bob' !== $name);
    }
);

// Returns false
$acl->isAllowed('manager', 'admin', 'dashboard');

$acl->setNoArgumentsDefaultAction(
    Acl::ALLOW
);

// Returns true
$acl->isAllowed('manager', 'admin', 'dashboard');
```

## Objects as role name and component name

Phalcon allows developers to define their own role and component objects. These objects must implement the supplied interfaces:

* [Phalcon\Acl\RoleAware](api/Phalcon_Acl_RoleAware) for Role
* [Phalcon\Acl\ComponentAware](api/Phalcon_Acl_ComponentAware) for Component

### Role

We can implement the [Phalcon\Acl\RoleAware](api/Phalcon_Acl_RoleAware) in our custom class with its own logic. The example below shows a new role object called `ManagerRole`:

```php
<?php

use Phalcon\Acl\RoleAware;

// Create our class which will be used as roleName
class ManagerRole implements RoleAware
{
    protected $id;

    protected $roleName;

    public function __construct($id, $roleName)
    {
        $this->id       = $id;
        $this->roleName = $roleName;
    }

    public function getId()
    {
        return $this->id;
    }

    // Implemented function from RoleAware Interface
    public function getRoleName()
    {
        return $this->roleName;
    }
}
```

### Component

We can implement the [Phalcon\Acl\ComponentAware](api/Phalcon_Acl_ComponentAware) in our custom class with its own logic. The example below shows a new role object called `ReportsComponent`:

```php
<?php

use Phalcon\Acl\ComponentAware;

// Create our class which will be used as componentName
class ReportsComponent implements ComponentAware
{
    protected $id;

    protected $componentName;

    protected $userId;

    public function __construct($id, $componentName, $userId)
    {
        $this->id            = $id;
        $this->componentName = $componentName;
        $this->userId        = $userId;
    }

    public function getId()
    {
        return $this->id;
    }

    public function getUserId()
    {
        return $this->userId;
    }

    // Implemented function from ComponentAware Interface
    public function getComponentName()
    {
        return $this->componentName;
    }
}
```

### ACL

These objects can now be used in our ACL.

```php
<?php

use ManagerRole;
use Phalcon\Acl;
use Phalcon\Acl\Adapter\Memory as AclList;
use Phalcon\Acl\Role;
use Phalcon\Acl\Component;
use ReportsComponent;

$acl = new AclList();

/**
 * Add the roles
 */
$acl->addRole('manager');

/**
 * Add the Components
 */
$acl->addComponent(
    'reports',
    [
        'list',
        'add',
        'view',
    ]
);

/**
 * Now tie them all together with a custom function. The ManagerRole and
 * ModelSbject parameters are necessary for the custom function to work 
 */
$acl->allow(
    'manager', 
    'reports', 
    'list',
    function (ManagerRole $manager, ModelComponent $model) {
        return boolval($manager->getId() === $model->getUserId());
    }
);

// Create the custom objects
$levelOne = new ManagerRole(1, 'manager-1');
$levelTwo = new ManagerRole(2, 'manager');
$admin    = new ManagerRole(3, 'manager');

// id - name - userId
$reports  = new ModelComponent(2, 'reports', 2);

// Check whether our user objects have access 
// Returns false
$acl->isAllowed($levelOne, $reports, 'list');

// Returns true
$acl->isAllowed($levelTwo, $reports, 'list');

// Returns false
$acl->isAllowed($admin, $reports, 'list');
```

The second call for `$levelTwo` evaluates `true` since the `getUserId()` returns `2` which in turn is evaluated in our custom function. Also note that in the custom function for `allow()` the objects are automatically bound, providing all the data necessary for the custom function to work. The custom function can accept any number of additional parameters. The order of the parameters defined in the `function()` constructor does not matter, because the objects will be automatically discovered and bound.

## Roles Inheritance

To remove duplication and increase efficiency in your application, the ACL offers inheritance in roles. This means that you can define one [Phalcon\Acl\Role](api/Phalcon_Acl_Role) as a base and after that inherit from it offering access to supersets or subsets of components. To use role inheritance, you need, you need to pass the inherited role as the second parameter of the method call, when adding that role in the list.

```php
<?php

use Phalcon\Acl;
use Phalcon\Acl\Adapter\Memory as AclList;
use Phalcon\Acl\Role;

$acl = new AclList();

/**
 * Create the roles
 */
$manager    = new Role('Managers');
$accounting = new Role('Accounting Department');
$guest      = new Role('Guests');

/**
 * Add the `guest` role to the ACL 
 */
$acl->addRole($guest);

/**
 * Add the `accounting` inheriting from `guest` 
 */
$acl->addRole($accounting, $guest);

/**
 * Add the `manager` inheriting from `accounting` 
 */
$acl->addRole($manager, $accounting);
```

Whatever access `guests` have will be propagated to `accounting` and in turn `accounting` will be propagated to `manager`

### Setup relationships after adding roles

Based on the application design, you might prefer to add first all the roles and then define the relationship between them.

```php
<?php

use Phalcon\Acl;
use Phalcon\Acl\Adapter\Memory as AclList;
use Phalcon\Acl\Role;

$acl = new AclList();

/**
 * Create the roles
 */
$manager    = new Role('Managers');
$accounting = new Role('Accounting Department');
$guest      = new Role('Guests');

/**
 * Add all the roles
 */
$acl->addRole($manager);
$acl->addRole($accounting);
$acl->addRole($guest);

/**
 * Add the inheritance 
 */
$acl->addInherit($manager, $accounting);
$acl->addInherit($accounting, $guest);

```

## Serializing ACL lists

[Phalcon\Acl](api/Phalcon_Acl) can be serialized and stored in a cache system to improve efficiency. You can store the serialized object in APC, session, file system, database, Redis etc. This way you can retrieve the ACL quickly without having to read the underlying data that create the ACL nor will you have to compute the ACL in every request.

```php
<?php

use Phalcon\Acl;
use Phalcon\Acl\Adapter\Memory as AclList;

$aclFile = 'app/security/acl.cache';
// Check whether ACL data already exist
if (true !== is_file($aclFile)) {

    // The ACL does not exist - build it
    $acl = new AclList();

    // ... Define roles, components, access, etc

    // Store serialized list into plain file
    file_put_contents(
        $aclFile,
        serialize($acl)
    );
} else {
    // Restore ACL object from serialized file
    $acl = unserialize(
        file_get_contents($aclFile)
    );
}

// Use ACL list as needed
if (true === $acl->isAllowed('manager', 'admin', 'dashboard')) {
    echo 'Access granted!';
} else {
    echo 'Access denied :(';
}
```

It is a good practice to not use serialization of the ACL during development, to ensure that your ACL is built in every request, while other adapters or means of serializing and storing the ACL in production.

## Events

[Phalcon\Acl](api/Phalcon_Acl) can work in conjunction with the [EventsManager](events) if present, to fire events to your application. Events are triggered using the type `acl`. Events that return `false` can stop the active role. The following events are available:

| Event Name          | Triggered                                                | Can stop role? |
| ------------------- | -------------------------------------------------------- |:--------------:|
| `afterCheckAccess`  | Triggered after checking if a role/component has access  |       No       |
| `beforeCheckAccess` | Triggered before checking if a role/component has access |      Yes       |

The following example demonstrates how to attach listeners to the ACL:

```php
<?php

use Phalcon\Acl;
use Phalcon\Acl\Adapter\Memory as AclList;
use Phalcon\Events\Event;
use Phalcon\Events\Manager as EventsManager;

// ...

// Create an event manager
$eventsManager = new EventsManager();

// Attach a listener for type 'acl'
$eventsManager->attach(
    'acl:beforeCheckAccess',
    function (Event $event, $acl) {
        echo $acl->getActiveRole() . PHP_EOL;

        echo $acl->getActiveComponent() . PHP_EOL;

        echo $acl->getActiveAccess() . PHP_EOL;
    }
);

$acl = new AclList();

// Setup the $acl
// ...

// Bind the eventsManager to the ACL component
$acl->setEventsManager($eventsManager);
```

## Implementing your own adapters

The [Phalcon\Acl\AdapterInterface](api/Phalcon_Acl_AdapterInterface) interface must be implemented in order to create your own ACL adapters or extend the existing ones.