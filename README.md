# Adic

**Another Dependency Injection Container for Unity 3D and beyond**

## Contents

* <a href="#introduction">Introduction</a>
* <a href="#features">Features</a>
* <a href="#concepts">Concepts</a>
	* <a href="#what-is">What is a dependency injector container?
	* <a href="#structure">Structure
	* <a href="#types-of-bindings">Types of bindings
	* <a href="#namespace-conventions">Namespace conventions
* <a href="#quick-start">Quick start</a>
* <a href="#api">API</a>
	* <a href="#bindings">Bindings</a>
	* <a href="#constructor-injection">Constructor injection</a>
	* <a href="#member-injection">Member injection</a>
	* <a href="#multiple-constructors">Multiple constructors</a>
	* <a href="#multiple-injection">Multiple injection</a>
	* <a href="#monobehaviour-injection">MonoBehaviour injection</a>
	* <a href="#conditions">Conditions</a>
	* <a href="#update">Update</a>
	* <a href="#dispose">Dispose</a>
	* <a href="#manual-type-resolution">Manual type resolution</a>
	* <a href="#factories">Factories</a>
	* <a href="#using-commands">Using commands</a>
* <a href="#order-of-events">Order of events
* <a href="#performance">Performance
* <a href="#container-extensions">Extensions</a>
	* <a href="#available-extensions">Available extensions</a>
		* <a href="#extension-bindings-printer">Bindings Printer</a>
		* <a href="#extension-commander">Commander</a>
		* <a href="#extension-context-root">Context Root</a>
		* <a href="#extension-event-caller">Event Caller</a>
		* <a href="#extension-mono-injection">Mono Injection</a>
		* <a href="#extension-unity-binding">Unity Binding</a>
	* <a href="#creating-extensions">Creating extensions</a>
	* <a href="#container-events">Container events</a>
		* <a href="#binder-events">Binder events</a>
		* <a href="#injector-events">Injector events</a>
* <a href="#general-notes">General notes</a>
* <a href="#examples">Examples</a>
* <a href="#support">Support</a>
* <a href="#license">License</a>

## <a id="introduction"></a>Introduction

*Adic* is a lightweight dependency injection container for Unity 3D.

Based on studies from [StrangeIoC](http://strangeioc.github.io/strangeioc/) and the proof of concept container from [Sebastiano Mandal�](http://blog.sebaslab.com/ioc-container-for-unity3d-part-1/), the ideia of the project was to create a dependency injection container that is simple to use and extend, having on its roots the simplicity of the work of Mandal� and the extensibility of StrangeIoC.

The project is compatible with Unity 3D 5 and 4 and possibly 3 (not tested) and should work on all available platforms (tested on Windows/Mac/Linux, Android, iOS and Web Player).

## <a id="features"></a>Features

* Bind types, singleton instances, factories, game objects and prefabs.
* Instance resolution by type, identifier and complex conditions.
* Injection on constructor, fields and properties.
* Can inject multiple objects of the same type.
* Can resolve and inject instances from types that are not bound to the container.
* Fast dependency resolution with internal cache.<a href=#performance>\*</a>
* Use of attributes to indicate injections, preferable construtors and post constructors.
* Can be easily extented through extensions.
* Framework decoupled from Unity - all Unity based API is achieved through extensions.
* Organized and well documented code written in C#.

## <a id="concepts"></a>Concepts

### <a id="what-is"></a>What is a dependency injection container?

A *dependency injection container* is a piece of software that handles the resolution of dependencies of objects. It's related to the [dependency injection](http://en.wikipedia.org/wiki/Dependency_injection) and [inversion of control](http://en.wikipedia.org/wiki/Inversion_of_control) design patterns.

The idea is that any dependency an object may need should be resolved by an external entity rather than the own object. Practically speaking, an object should not use `new` to create the objects it uses, having those instances *injected* into it by another object whose sole existence is to resolve dependencies.

So, a *dependency injection container* holds information about dependencies (the *bindings*) that can be injected into another objects by demand (injecting into existing objects) or during resolution (when you are creating a new object of some type).

### <a id="structure"></a>Structure

The structure of *Adic* is divided into five parts:

1. **InjectionContainer/Container**: binds, resolves, injects and holds dependencies. Technically, the container is a *Binder* and an *Injector* at the same time.
2. **Binder**: binds a type to another type or instance with inject conditions.
3. **Injector**: resolves and injects dependencies.
4. **Context Root**: main context in which the containers are in. Acts as an entry point for the game. It's implemented through an <a href="#extension-context-root">extension</a>.
5. **Extensions**: provides additional features to the framework.

### <a id="types-of-bindings"></a>Types of bindings

* **Transient**: a new instance is created each time a dependency needs to be resolved.
* **Singleton**: a single instance is created and used on any dependency resolution.
* **Factory**: creates the instance and returns it to the container.

### <a id="namespace-conventions"></a>Namespace conventions

*Adic* is organized internally into different namespaces that representes the framework components. However, the components commonly used are under `Adic` namespace:

1. Attributes (`Inject`, `Construct`, `PostConstruct`);
2. `InjectionContainer`;
3. `IFactory`;
4. Extensions (like `ContextRoot` and `UnityBinding`).

## <a id="quick-start"></a>Quick start

1\. Create the context root (e. g. GameRoot.cs) of your scene by inheriting from `Adic.ContextRoot`.

**Note**: there should be only one context root per scene.

**Hint**: when using a context root for each scene of your game, to make the project more organized, on `Scripts` folder create folders for each of your scenes that will hold their own scripts and context root.
   
```cs
using UnityEngine;

namespace MyNamespace {
	/// <summary>
	/// Game context root.
	/// </summary>
	public class GameRoot : Adic.ContextRoot {
		public override void SetupContainers() {
			//Setup the containers.
		}

		public override void Init() {
			//Init the game.
		}
	}
}
```
   
2\. On `SetupContainers()` method, create and add any containers will may need.

```cs
public override void SetupContainers() {
	//Create a container.
	var container = new Adic.InjectionContainer();

	//Setup bindinds.
	container.Bind<Whatever>().ToSelf();

	//Add the container to the context.
	this.AddContainer(container);
}
```

**Hint**: on *Adic* the lifetime of your bindings is the lifetime of your containers. So, you can create as many containers as you want to hold your dependencies.

3\. On the `Init()` method, place any code to start your game.

**Note**: the idea of this method is to work as an entry point for your game, like a `main()` method on console applications.

4\. Attach the context root created by you on an empty game object in your scene.

5\. Start dependency injecting!

## <a id="api"></a>API

### <a id="bindings"></a>Bindings

Binding is the action of linking a type to another type or instance. *Adic* makes it simple by providing different ways to create your bindings.

Every binding must occur on a certain key type by calling the `Bind()` method of the container. 

The simple way to bind e.g. some interface to its class implementation is as below:
   
```cs
container.Bind<SomeInterface>().To<ClassImplementation>();
```

It's also possible to bind a class to an existing instance:

```cs
container.Bind<SomeInterface>().To(someInstance);
```

You can also bind a Unity component to a game object that has that particular component:

```cs
container.Bind<Transform>().ToGameObject("GameObjectNameOnHierarchy");
```

Or a prefab on some `Prefabs/Whatever` resources folder:

```cs
container.Bind<Transform>().ToPrefab("Prefabs/Whatever/MyPrefab");
```

And, if needed, non generics versions of bindings' methods are also available:

```cs
container.Bind(someType).To(anotherType);
```

The next sections will cover all the available bindings *Adic* provides.

#### To Self

Binds the key type to a transient of itself. The key must be a class.

```cs
container.Bind<ClassType>().ToSelf();
```

#### To Singleton

Binds the key type to a singleton of itself. The key must be a class.

```cs
container.Bind<ClassType>().ToSingleton();
```

It's also possible to create a singleton of the key type to another type. In this case, the key may not be a class.

```cs
//Using generics...
container.Bind<InterfaceType>().ToSingleton<ClassType>();
//...or instance type.
container.Bind<InterfaceType>().ToSingleton(classTypeObject);
```

#### To another type

Binds the key type to a transient of another type. In this case, the *To* type will be instantiated every time a resolution of the key type is asked.

```cs
//Using generics...
container.Bind<InterfaceType>().To<ClassType>();
//..or instance type.
container.Bind<InterfaceType>().To(classTypeObject);
```

#### To instance

Binds the key type to an instance.

```cs
//Using generics...
container.Bind<InterfaceType>().To<ClassType>(instanceOfClassType);
//..or instance type.
container.Bind<InterfaceType>().To(classTypeObject, instanceOfClassType);
```

#### To a Factory

Binds the key type to a factory. The factory must implement the `Adic.IFactory` interface.

```cs
container.Bind<InterfaceType>().ToFactory(factoryInstance);
```

See <a href="#factories">Factories</a> for more information.

#### To game object

Binds the key type to a singleton of itself or some type on a new game object.

```cs
//Binding to itself...
container.Bind<SomeMonoBehaviour>().ToGameObject();
//...or some other component using generics...
container.Bind<SomeInterface>().ToGameObject<SomeMonoBehaviour>();
//..or some other component by instance type.
container.Bind<SomeInterface>().ToGameObject(someMonoBehaviourType);
```

The newly created game object will have the same name as the key type.

#### To game object by name

Binds the key type to a singleton `UnityEngine.Component` of itself or some type on a game object of a given name.

If the component is not found on the game object, it will be added.

```cs
//Binding to itself by name...
container.Bind<SomeMonoBehaviour>().ToGameObject("GameObjectName");
//...or some other component using generics and name...
container.Bind<SomeInterface>().ToGameObject<SomeMonoBehaviour>("GameObjectName");
//..or some other component by instance type and name.
container.Bind<SomeInterface>()().ToGameObject(someMonoBehaviourType, "GameObjectName");
```

#### To game object with tag

Binds the key type to a singleton `UnityEngine.Component` of itself or some type on a game object of a given tag.

If the component is not found on the game object, it will be added.

```cs
//Binding to itself by tag...
container.Bind<SomeMonoBehaviour>().ToGameObjectWithTag("Tag");
//...or some other component using generics and tag...
container.Bind<SomeInterface>().ToGameObjectWithTag<SomeMonoBehaviour>("Tag");
//..or some other component by instance type and tag.
container.Bind<SomeInterface>().ToGameObjectWithTag(someMonoBehaviourType, "Tag");
```

#### To prefab transient

Binds the key type to a transient `UnityEngine.Component` of itself or some type on the prefab.

If the component is not found on the game object, it will be added.

```cs
//Binding prefab to itself...
container.Bind<SomeMonoBehaviour>().ToPrefab("Prefabs/Whatever/MyPrefab");
//...or to another component on the prefab using generics...
container.Bind<SomeInterface>().ToPrefab<SomeMonoBehaviour>("Prefabs/Whatever/MyPrefab");
//...or to another component on the prefab using instance tyoe.
container.Bind<SomeInterface>().ToPrefab(someMonoBehaviourType, "Tag");
```

#### To prefab singleton

Binds the key type to a singleton `UnityEngine.Component` of itself or some type on a newly instantiated prefab.

```cs
//Binding singleton prefab to itself...
container.Bind<SomeMonoBehaviour>().ToPrefabSingleton("Prefabs/Whatever/MyPrefab");
//...or to another component on the prefab using generics...
container.Bind<SomeInterface>().ToPrefabSingleton<SomeMonoBehaviour>("Prefabs/Whatever/MyPrefab");
//...or to another component on the prefab using instance tyoe.
container.Bind<SomeInterface>().ToPrefabSingleton(someMonoBehaviourType, "Tag");
```

### <a id="constructor-injection"></a>Constructor injection

*Adic* will always try to resolve any dependencies the constructor may need by using information from its bindings or trying to instantiate any types that are unknown to the binder.

**Note 1**: if there's more than one constructor, *Adic* always look for the one with less parameteres. However, <a href="#multiple-constructors">it's possible to indicate which constructor will be used</a> on a multi constructor class.

**Note 2**: there's no need to decorate constructors' parameteres with `Inject` attributes.

**Note 3**: currently, injection identifiers are not supported on construtors. However, <a href="#conditions">any conditions</a> (that are not identifiers) on types are also applied to the constructor parameters.

### <a id="member-injection"></a>Member injection

*Adic* car perform dependency injection on public fields and properties of classes. To make it happen, just decorate the members with the `Inject` attribute:

```cs
namespace MyNamespace {
	/// <summary>
	/// My class summary.
	/// </summary>
	public class MyClass {
		/// <summary>Field to be injected.</summary>
		[Inject]
		public SomeClass fieldToInject;
		/// <summary>Field NOT to be injected.</summary>
		public SomeClass fieldNotToInject;

		/// <summary>Property to be injected.</summary>
		[Inject]
		public SomeOtherClass propertyToInject { get; set; }		
		/// <summary>Property NOT to be injected.</summary>
		public SomeOtherClass propertyNotToInject { get; set; }
	}
}
```

If you need to perform actions after all the injections took place, create a method and decorate it with the `PostConstruct` attribute:

```cs
namespace MyNamespace {
	/// <summary>
	/// My class summary.
	/// </summary>
	public class MyClass {
		/// <summary>Field to be injected.</summary>
		[Inject]
		public SomeClass fieldToInject;

		/// <summary>
		/// Class constructor.
		/// </summary>
		public MyClass() {
			...
		}

		/// <summary>
		/// Class post constructor, called after all the dependencies have been resolved.
		/// </summary>
		[PostConstruct]
		public void PostConstruct() {
			...
		}
	}
}
```

### <a id="multiple-constructors"></a>Multiple constructors

In case you have multiple constructors, it's possible to indicate to *Adic* which one should be used by decorating it with the `Construct` attribute:

```cs
namespace MyNamespace {
	/// <summary>
	/// My class summary.
	/// </summary>
	public class MyClass {
		/// <summary>
		/// Class constructor.
		/// </summary>
		public MyClass() {
			...
		}

		/// <summary>
		/// Class constructor.
		/// </summary>
		/// <param name="parameterName">Parameter description</param>
		[Construct]
		public MyClass(Type parameterName) {
			...
		}
	}
}
```

### <a id="multiple-injection"></a>Multiple injection

It's possible to inject multiple objects of the same type by creating a series of bindings of the same key type. In this case, the injection occurs on an array of the key type.

Binding multiple objects to the same key:

```cs
container.Bind<GameObject>().ToGameObject("Enemy1");
container.Bind<GameObject>().ToGameObject("Enemy2");
container.Bind<GameObject>().ToGameObject("Enemy3");
container.Bind<GameObject>().ToGameObject("Enemy4");
container.Bind<GameObject>().ToGameObject("Enemy5");
```

Multiple injection on a field:

```cs
namespace MyNamespace {
	/// <summary>
	/// My class summary.
	/// </summary>
	public class MyClass {
		/// <summary>Enemies on the game.</summary>
		[Inject]
		public GameObject[] enemies;
	}
}
```

It's possible to manually resolve multiple objects. Please see <a href="#manual-type-resolution">Manual type resolution</a> for more information.

### <a id="monobehaviour-injection"></a>MonoBehaviour injection

It's possible to perform injection on custom MonoBehaviour fields and properties by using the <a href="#extension-mono-injection">Mono Injection</a> extension, which is enabled by default, by calling `this.Inject()` on the `Start()` method of the MonoBehaviour:

```cs
using Unity.Engine;

namespace MyNamespace {
	/// <summary>
	/// My MonoBehaviour summary.
	/// </summary>
	public class MyBehaviour : MonoBehaviour {
		/// <summary>Field to be injected.</summary>
		[Inject]
		public SomeClass fieldToInject;

		protected void Start() {
			this.Inject();
		}
	}
}
```

To make injection even simpler, create a base behaviour from which all your MonoBehaviour will inherit:

```cs
using Unity.Engine;

namespace MyNamespace {
	/// <summary>
	/// Base MonoBehaviour.
	/// </summary>
	public abstract class BaseBehaviour : MonoBehaviour {
		/// <summary>
		/// Called when the component is starting.
		/// 
		/// If overriden on derived classes, always call base.Start().
		/// </summary>
		protected virtual void Start() {
			this.Inject();
		}
	}
}
```

### <a id="conditions"></a>Conditions

Conditions allow a more customized approach when injecting dependencies into constructors and fields/properties.

Using conditions you can:

1\. Tag a binding with an identifier, so you can indicate it as a parameter in the `Inject` attribute on fields/properties:

When binding:

```cs
container.Bind<SomeInterface>().To<SomeClass>().As("Identifier");
```

When injecting:

```cs
namespace MyNamespace {
	/// <summary>
	/// My class summary.
	/// </summary>
	public class MyClass {
		/// <summary>Field to be injected.</summary>
		[Inject("Identifier")]
		public SomeInterface field;
	}
}
```

2\. Indicate in which objects a binding can be injected, by type or instance:

```cs
//Using generics...
container.Bind<SomeInterface>().To<SomeClass>().WhenInto<MyClass>();
//...or type instance...
container.Bind<SomeInterface>().To<SomeClass>().WhenInto(myClassInstanceType);
//...or by a given instance.
container.Bind<SomeInterface>().To<SomeClass>().WhenIntoInstance(myClassInstanceType);
```

3\. Create complex conditions by using an anonymous method:

```cs
container.Bind<SomeInterface>().To<SomeClass>().When(context => 
		context.member.Equals(InjectionMember.Field) &&
        context.parentType.Equals(typeof(MyClass))
	);
```

The context provides the following fields:

1. **member** (`Adic.InjectionMember` enum): the class member in which the injection is occuring (*None*, *Constructor*, *Field* or *Property*).
2. **memberType** (`System.Type`): the type of the member in which the injection is occuring.
3. **identifier** (`object`): the identifier of the member in which the injection is occuring (from `InjectAttribute`).
4. **parentType** (`object`): the type of the object in which the injection is occuring.
5. **parentInstance** (`object`): the instance of the object in which the injection is occuring.
6. **injectType** (`System.Type`): the type of the object being injected.

### <a id="update"></a>Update

It's possible to have an `Update()` method on regular classes (that don't inherit from `UnityEngine.MonoBehaviour`) by implementing the `Adic.IUpdatable` interface.

See <a href="#extension-event-caller">Event Caller</a> for more information.

### <a id="dispose"></a>Dispose

When a scene is destroyed, it's possible to have a method that can be called to e.g. free up resources. To do it, implement the `System.IDisposable` interface on any class that you want to have this option.

See <a href="#extension-event-caller">Event Caller</a> for more information.

### <a id="manual-type-resolution"></a>Manual type resolution

If you need to get a type from the container but do not want to use injection through constructor or fields/properties, it's possible to execute a manual resolution directly by calling the `Resolve()` method:

```cs
//Resolving using generics...
var instance = container.Resolve<Type>();
//...or by type instance.
instance = container.Resolve(typeInstance);
```

It's also possible to resolve all objects of a given type:

```cs
//Resolving all objects using generics...
var instances = container.ResolveAll<Type>();
//...or by type instance.
instances = container.ResolveAll(typeInstance);
```

**Note**: currently manual resolution of bindings that has conditions is not supported.

### <a id="factories"></a>Factories

When you need to handle the instantiation of an object manually, it's possible to create a factory class by inheriting from `Adic.IFactory`:

```cs
namespace MyNamespace {
	/// <summary>
	/// My factory.
	/// </summary>
	public class MyFactory : Adic.IFactory {
		/// <summary>Type the factory creates.</summary>
		Type factoryType { 
			get { return typeof(FactoryObjectType); } 
		}

		/// <summary>
		/// Creates an instance of the object of the type created by the factory.
		/// </summary>
		/// <returns>The instance.</returns>
		public object Create() {
			...
		}
	}
}
```

### <a id="using-commands"></a>Using commands

#### What are commands?

Commands are actions executed by your game, usually in response to an event.

The concept of commands is to place everything the action/event needs in a single place, so it's easy to understand and maintain it.

Suppose you have an event of enemy destroyed. When that occurs, you have to update UI, dispose the enemy, spawn some particles and save statistics somewhere. One approach would be dispatch the event to every object that has to do something about it, which is fine given it keeps single responsibility by allowing every object to take care of their part on the event. However, when your project grows, it can be a nightmare to find every place a given event is handled. That's when commands come in handy: you place all the code and dependencies for a certain action/event in a single place.

#### Creatings commands

To create a command, inherit from `Adic.Command` and override the `Execute()` method, where you will place all the code needed to execute the command. If you have any dependencies to be injected before executing the command, add them as fields or properties and decorate them with an `Inject` attribute:

```cs
namespace MyNamespace.Commands {
	/// <summary>
	/// My command.
	/// </summary>
	public class MyCommand : Adic.Command {
		/// <summary>Some dependency to be injected.</summary>
		[Inject]
		public object someDependency;

		/// <summary>
		/// Executes the command.
		/// <param name="parameters">Command parameters.</param>
		public override void Execute(params object[] parameters) {
			//Execution code.
		}
	}
}
```
**Hint**: it's also possible to wire any dependencies through constructor. However, in this case the dependencies will only be resolved once, during instantiation.

**Note**: it's a good practice to place all your commands under the same namespace, so it's easy to register them.

##### Types of commands

###### Pooled

The command is kept in a pool for reuse, avoiding new instantiations. It's useful for commands that need to maintain state when executing. This is the default behaviour.

When creating pooled commands, it's possible to set the initial and maximum size of the pool for a particular command by setting, respectively, the `preloadPoolSize` and `maxPoolSize` properties:

```cs
namespace MyNamespace.Commands {
	/// <summary>
	/// My command.
	/// </summary>
	public class MyCommand : Adic.Command {
		/// <summary>The quantity of the command to preload on pool (default: 1).</summary>
		public override int preloadPoolSize { get { return 5; } }
		/// <summary>The maximum size pool for this command (default: 10).</summary>
		public override int maxPoolSize { get { return 20; } }

		/// <summary>
		/// Executes the command.
		/// <param name="parameters">Command parameters.</param>
		public override void Execute(params object[] parameters) {
			//Execution code.
		}
	}
}
```

###### Singleton

There's only one copy of the command available, which is used every time the command is dispatched. It's useful for commands that don't need state or every dependency the command needs is given during execution. To make a command singleton, return `true` in the `singleton` property of the command:

```cs
namespace MyNamespace.Commands {
	/// <summary>
	/// My command.
	/// </summary>
	public class MyCommand : Adic.Command {
		/// <summary>Indicates whether this command is a singleton (there's only one instance of it).</summary>
		public override bool singleton { get { return true; } }

		/// <summary>
		/// Executes the command.
		/// <param name="parameters">Command parameters.</param>
		public override void Execute(params object[] parameters) {
			//Execution code.
		}
	}
}
```

**Note**: When using singleton commands, injection is done only through constructors or injection after command instantiation.

#### Registering commands

To register a command, call the `Register()` method on the container, usually on the context root:

```cs
using UnityEngine;

namespace MyNamespace {
	/// <summary>
	/// Game context root.
	/// </summary>
	public class GameRoot : Adic.ContextRoot {
		public override void SetupContainers() {
			//Create the container.
			var container = new InjectionContainer();
			//Register any extensions the container may use.
			container.RegisterExtension<CommanderContainerExtension>();

			//Registering by generics...
			container.RegisterCommand<MyCommand>();
			//...or by type.
			container.RegisterCommand(typeof(MyCommand));
		}

		public override void Init() {
			//Init the game.
		}
	}
}
```

It's also possible to register all commands under the same namespace by calling the `RegisterCommands()` method on the container and passing the full name of the namespace:

```cs
public override void SetupContainers() {
	//Create the container.
	var container = new InjectionContainer();
	//Register any extensions the container may use.
	container.RegisterExtension<CommanderContainerExtension>();

	//Register all commands under the namespace "MyNamespace.Commands".
	container.RegisterCommands("MyNamespace.Commands");
}
```

**Note**: when registering a command, it's placed on the container, so it's easier to resolve it and its dependencies.

#### Dispatching commands

To dispatch a command, just call the `Dispatch()` method on the `Adic.CommandDispatcher`, using either the generics or the by `System.Type` versions:

```cs
/// <summary>
/// My method that dispatches a command.
/// </summary>
public void MyMethodThatDispatchesCommands() {
	//Dispatching by generics...
	dispatcher.Dispatch<MyCommand>();
	//...or by type.
	dispatcher.Dispatch(typeof(MyCommand));
}
```

To use the `CommandDispatcher`, you have to inject it wherever you need to use it:

```cs
namespace MyNamespace {
	/// <summary>
	/// My class that dispatches commands.
	/// </summary>
	public class MyClassThatDispatchesCommands {
		/// <summary>The command dispatcher.</summary>
		[Inject]
		public CommandDispatcher dispatcher;

		/// <summary>
		/// My method that dispatches a command.
		/// </summary>
		public void MyMethodThatDispatchesCommands() {
			this.dispatcher.Dispatch<MyCommand>();
		}
	}
}
```

**Note 1**: when dispatching a command, it's placed in a list in the `CommandDispatcher`, which is the one responsible for pooling and managing existing commands.

**Note 2**: commands in the pool that are not singleton are *reinjected* every time they are executed.

## <a id="order-of-events"></a>Order of events

1. Unity Awake()
2. ContextRoot calls SetupContainers()
3. ContextRoot asks for each container to generate cache for its types
4. ContextRoot calls Init()
5. Unity Start() on all MonoBehaviours
6. Injection on MonoBehaviours
7. Update() is called on every object that implemented `Adic.IUpdatable`
8. Scene is destroyed
9. Dispose() is called on every object that implemented `System.IDisposable`

## <a id="performance"></a>Performance

*Adic* was created with speed in mind, using internal cache to minimize the use of [reflection](http://en.wikipedia.org/wiki/Reflection_%28computer_programming%29) (which is usually slow), ensuring a good performance when resolving and injecting into objects - the container can resolve a 1.000 objects in 0.002s and a 1.000.000 in 2s<a href="#about-performance-tests">\*</a>.

To maximize performance, always bind all types that will be resolved/injected on the <a href="#quick-start">ContextRoot</a>, so *Adic* can generate cache of the objects and use that information during runtime.

If you have more than one container on the same scene, it's possible to share the cache between them. To do so, create an instance of `Adic.Cache.ReflectionCache` and pass it to any container you create:

```cs
using UnityEngine;

namespace MyNamespace {
	/// <summary>
	/// Game context root.
	/// </summary>
	public class GameRoot : Adic.ContextRoot {
		public override void SetupContainers() {
			//Create the reflection cache.
			var cache = new ReflectionCache();

			//Create a new container.
			var container1 = new InjectionContainer(cache);

			//Container bindings...

			//Create a new container.
			var container2 = new InjectionContainer(cache);

			//Container bindings...
		}

		public override void Init() {
			//Init the game.
		}
	}
}
```

<sup><a id="about-performance-tests">\*</a> See *Tests/Editor/SpeedTest.cs* for more details on performance tests. Tested on a MacBook Pro late 2014 (i7 2.5/3.7 GHz).</sup>

<sup>\- 1 thousand simple resolves in 00:00:00.0023210s</sup><br>
<sup>\- 1 million simple resolves in 00:00:02.3380960s</sup><br>
<sup>\- 1 thousand more complex resolves in 00:00:00.0045590s</sup><br>
<sup>\- 1 million more complex resolves in 00:00:04.8917720s</sup>

## <a id="container-extensions"></a>Extensions

Extensions are a way to enhance *Adic* without having to edit it to suit different needs. By using extensions, the core of *Adic* is kept agnostic, so it can be used on any C# environment.

## <a id="available-extensions"></a>Available extensions

### <a id="extension-bindings-printer"></a>Bindings Printer

Prints all bindings on any containers on the current `ContextRoot`. It must be executed on Play Mode.

To open the Bindings Printer windows, click on *Windows/Adic/Bindings Printer* menu.

#### Format

```
[Container Type Full Name] (index: [Container Index on ContextRoot], [destroy on load/singleton])

	[For each binding]
	Type: [Binding Type Full Name]
	Bound to: [Bound To Type Full Name] ([type/instance])
	Binding type: [Transient/Singleton/Factory]
	Identifier [Identifier/-]
	Conditions: [yes/no]
```

#### Dependencies

* <a href="#extension-context-root">Context Root</a>

### <a id="extension-commander"></a>Commander

Provides dispatching of commands, with pooling for better performance.

For more information on commands, see <a href="#using-commands">Using commands</a>.

#### Configuration

Register the extension on any containers that will use it:

```cs
//Creates the container.
var container = new InjectionContainer();
//Register any extensions the container may use.
container.RegisterExtension<CommanderContainerExtension>();
```

If you use `IDiposable` or `IUpdatable` events, also register the <a href="#extension-event-caller">Event Caller</a> extension:

```cs
//Creates the container.
var container = new InjectionContainer();
//Register any extensions the container may use.
container.RegisterExtension<CommanderContainerExtension>();
container.RegisterExtension<EventCallerContainerExtension>();
```

#### Dependencies

* <a href="#extension-event-caller">Event Caller</a> (only if using `IDiposable` or `IUpdatable` events)

### <a id="extension-context-root"></a>Context Root

Provides an entry point for the game on Unity 3D.

#### Configuration

Please see <a href="#quick-start">Quick start</a> for more information.

#### Notes

1. When adding containers using `AddContainer()`, it's possible to keep them alive between scenes by setting the `destroyOnLoad` to `false`.

#### Dependencies

None

### <a id="extension-event-caller"></a>Event Caller

Calls events on classes that implement certain interfaces. The classes must be bound to a container.

#### Available events

##### Update

Calls `Update()` method on classes that implement `Adic.IUpdatable` interface.

```cs
namespace MyNamespace {
	/// <summary>
	/// My updateable class.
	/// </summary>
	public class MyUpdateableClass : Adic.IUpdatable {
		public void Update() {
			//Update code.
		}
	}
}
```

##### Dispose

When a scene is destroyed, calls `Dispose()` method on classes that implement `System.IDisposable` interface.

```cs
namespace MyNamespace {
	/// <summary>
	/// My disposable class.
	/// </summary>
	public class MyDisposableClass : System.IDisposable {
		public void Dispose() {
			//Dispose code.
		}
	}
}
```

#### Configuration

Register the extension on any containers that will use it:

```cs
//Creates the container.
var container = new InjectionContainer();
//Register any extensions the container may use.
container.RegisterExtension<EventCallerContainerExtension>();
```

#### Notes

1. Currently, any objects that are updateable are not removed from the update's list when they're not in use anymore. So, it's recommended to implement the `Adic.IUpdatable` interface only on singleton or transient objects that will live until the scene is destroyed;
2. When the scene is destroyed, the update's list is cleared. So, any objects that will live between scenes that implement the `Adic.IUpdatable` interface will not be readded to the list. **It's recommeded to use updateable objects only on the context of a single scene**.

#### Dependencies

None

### <a id="extension-mono-injection"></a>Mono Injection

Allows injection on MonoBehaviours by provinding an `Inject` method to `UnityEngine.MonoBehaviour`.

#### Configuration

Please see <a href="#monobehaviour-injection">MonoBehaviour injection</a> for more information.

#### Dependencies

* <a href="#extension-context-root">Context Root</a>

### <a id="extension-unity-binding"></a>Unity Binding

Provides Unity 3D bindings to the container.

Please see <a href="#bindings">Bindings</a> for more information.

#### Configuration

Register the extension on any containers that you may use it:

```cs
//Creates the container.
var container = new InjectionContainer();
//Register any extensions the container may use.
container.RegisterExtension<UnityBindingContainerExtension>();
```

#### Notes

1. **ALWAYS CALL `Inject()` FROM 'Start'**! (use the <a href="#extension-mono-injection">Mono Injection</a> Extension).

#### Dependencies

None

## <a id="creating-extensions"></a>Creating extensions

Extensions on *Adic* can be created in 3 ways:

1. Creating a framework extension extending the base APIs through their interfaces;
2. Creating extension methods to any part of the framework;
3. Creating a container extension, which allows for the interception of internal events, which can alter the inner workings of the framework.

**Note**: always place the public parts of extensions into *Adic* namespace.

To create a *container extension*, which can intercept internal *Adic* events, you have to:

1\. Create the extension class with `ContainerExtension` sufix.

2\. Implement `Adic.Container.IContainerExtension`.

3\. Subscribe to any events on the container on OnRegister method.

```cs
public void OnRegister(IInjectionContainer container) {
	container.beforeAddBinding += this.OnBeforeAddBinding;
}
```

4\. Unsubscribe to any events the extension may use on the container on OnUnregister method.

```cs
public void OnUnregister(IInjectionContainer container) {
	container.beforeAddBinding -= this.OnBeforeAddBinding;
}
```

## <a id="container-events"></a>Container events

Container events provide a way to intercept internal actions of the container and change its inner workings to suit the needs of your extension.

All events are available through `Adic.InjectionContainer`.

### <a id="binder-events"></a>Binder events

* `beforeAddBinding`: occurs before a binding is added.
* `afterAddBinding`: occurs after a binding is added.
* `beforeRemoveBinding`: occurs before a binding is removed.
* `afterRemoveBinding`: occurs after a binding is removed.

### <a id="injector-events"></a>Injector events

* `beforeResolve`: occurs before a type is resolved.
* `afterResolve`: occurs after a type is resolved.
* `bindingEvaluation`: occurs when a binding is available for resolution.
* `bindingResolution`: occures when a binding is resolved to an instance.
* `beforeInject`: occurs before an instance receives injection.
* `afterInject`: occurs after an instance receives injection.

## * <a id="general-notes">General notes</a>

1. If an instance is not found, it will be resolved to NULL;
2. Multiple injections should occur on an array of the desired type;
3. Order of bindings is controlled by just reordering the bindings;
4. Adic relies on Unity Test Tools for unit testing. You can download it at [Unity Asset Store](https://www.assetstore.unity3d.com/#!/content/13802).

## <a id="examples"></a>Examples

There are some examples that are bundled to the main package that teach the basics and beyond of *Adic*.

### 1. Hello World

Exemplifies the basics of how to setup a scene for dependency injection using the ContextRoot.

### 2. Binding Game Objects

Exemplifies how to bind components to new and existing game objects and allows them to share dependencies.

### 3. Using conditions

Exemplifies the use of condition identifiers on injections.

### 4. Prefabs

Exemplifies how to bind to prefabs and the use of `PostConstruct` as a second constructor.

### 5. Commander

Exemplifies the use of commands through a simple spawner of a prefab.

## <a id="support"></a>Support

Found a bug? Please create an issue on the [GitHub project page](https://github.com/intentor/adic/) or send a pull request if have a fix or extension.

You can also send me a message at support@intentor.com.br to discuss more obscure matters about *Adic*.

## <a id="license"></a>License

Licensed under the [The MIT License (MIT)](http://opensource.org/licenses/MIT). Please read LICENSE for more information.