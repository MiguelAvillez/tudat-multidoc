# TUDAT & TUDATPY API

This is an instructional document including a template for documenting enums, classes and (factory) functions.


## Table of Contents
1. Basic Concepts
2. Tudat vs. Tudatpy
3. Documentation style
4. Linking docstrings to the code
5. Linking within API
6. Linking external resources
7. Modifying exposure code 
8. Full template
   
## Basic Concepts

Docstrings are kept in yaml files in the tudat-bundle/tudat-multidoc/docstrings directory.
The content is divided over a file tree structure that mimics the structure of the tudatpy exposure in
tudat-bundle/tudatpy/tudatpy/kernel, which is the same structure of the tudatpy modules.
Each file bundles the content of a module exposure function (i.e. Ephemeris, Gravity Field, Rotation, etc). Within each
yaml file, all module classes are listed under a single "classes:" key, while functions are listed under a single "functions" key.


## TUDAT vs. TUDATPY

Tudat and tudatpy API documentations are generated from the same yaml file.

Tudat-exclusive content is marked by the `# [cpp]` tag, while tudatpy-exclusive content is marked by the `# [py]`.
Untagged content will be included in both API documentations.

Typically, the two APIs convey the same content. That means that the same functions, parameters and returns (etc) are
listed in both APIs, where names and types are adopted to the respective API (`[cpp]` or `[py]`).
Most class or function summaries are the same (word-by-word) for the two APIs.

An exception to the analogous structure of the two APIs is the treatment of class attributes:
Class attributes are only documented as such for the tudatpy API, while the tudat API documents the get/set
methods of the classes instead.
This is of course reflected in the exposure of classes in tudatpy, where "properties" (attributes) of the
tudatpy classes are exposed by making them available via the original get/set methods of the tudat classes.


## Documentation style: classes and related factory functions

Factory functions (FF, functions creating instances of objects via the class constructors ) are intended to be the
user's interface with the actual class constructors, such that the users typically do not interact with the
classes as such. FF's will be used throughout all user guides, examples and tutorials. They will be the user`s
landing pad in the API. It is therefore the intention to supply all functionality-related information in the
docstrings of the FF - this may include (but is not limited to) complete explanations for function parameters,
information about the models (that will be created by the classes), model implementation and links to external
resources.

Classes on the other hand are documented in a more minimalistc manner, that is focussed on code design, hierachy
and less the functional aspects. Constructors of classes that have FF's implemented will not be documented with
`parameters` and `returns` keys, since users are discouraged from directly using the constructor method.
`short_description` of the constructor method will be given by the string "Constructor.
`extended_description` of the constructor method will refer the user to use the respective FF for creating
instances of the given class.
Base classes are to be identified as such (in `short_description`). Typically, users do not create instances of the
base classes (but of the derived classes through the dedicated FFs) and this shall also be mentioned in the
in the `extended_description`.


## Linking docstrings to respective function and classes

The docstrings need to be linked in the code, such that during the API build a connection from docstrings to the code can be made.

### TUDAT

This is done by placing tags right above the class/function declaration in the header files of the cpp source code (tudat-bundle/tudat/include/):
- for classes:
  ````
  //! @get_docstring(<ClassName>.__docstring__)
  ````
- for functions:
  ````
  //! @get_docstring(<function_name>)
  ````
- for overload nr X (X=0,1,...) of a function:
  ````
  //! @get_docstring(<function_name>, X)
  ````

> **Note:** all other tags present in .cpp/.h files, used to connect the source code to the Doxygen documentation engine,
> should be removed, otherwise they will be automatically included in the API.

### TUDATPY

In order to make the link to the exposed tudatpy classes and functions, the docstring needs to be exposed as well:
- for classes:
  ````
  get_docstring("<ClassName>").c_str()
  ````
  as last argument of `py:class_<>()`, as in
  ````
  py:class_<CppClass, CppPointerToClass, CppParentClass>(module, "ClassName", get_docstring("<ClassName>").c_str())
  ````
- for class methods and properties (generically "fields"):
  ````
  get_docstring("<ClassName.FieldName>").c_str()
  ````
  - for class methods: as last argument of `.def()`, as in
    ````
    .def("MethodName", CppClassName::CppMethodName, py::arg("ParameterName"), ..., get_docstring("<ClassName.MethodName>").c_str())
    ````
  - for properties: as last argument of `.def_property()` (or `.def_property_readonly()` for properties with a getter only), as in
    ````
    .def_property("PropertyName", CppClassName::CppGetterMethodName, CppClassName::CppSetterMethodName, get_docstring("<ClassName.PropertyName>").c_str())
    ````
    or
    ````
    .def_property_readonly("PropertyName", CppClassName::CppGetterMethodName, get_docstring("<ClassName.PropertyName>").c_str())
    ````
- for free functions:
  ````
  get_docstring("<function_name>").c_str()
  ````
  as last argument of `m.def("<function_name>", ... )` exposure function
- for overload nr X (X=0,1,...) of a function:
  ````
  get_docstring("<function_name>", X).c_str()
  ````
  as last argument of `m.def("<function_name>", ... )` exposure function


**Note**: class attributes do not need the get_docstring tag, because their docstring is automatically retrieved from
the class exposure.

### Examples: TUDAT

#### TUDAT class
 ````c++
 //! @get_docstring(ThrustAccelerationSettings.__docstring__)
 class ThrustAccelerationSettings: public AccelerationSettings{
 ...
 }
 ````

#### TUDAT regular function
````c++
 //! @get_docstring(customAccelerationSettings)
 inline std::shared_ptr< AccelerationSettings > customAccelerationSettings(
         const std::function< Eigen::Vector3d( const double ) > accelerationFunction,
         const std::function< double( const double ) > scalingFunction = nullptr )
 {
 ...
 }
````

#### TUDAT overloaded function
````c++
 //! @get_docstring(thrustAcceleration, 0)
 inline std::shared_ptr< AccelerationSettings > thrustAcceleration( const std::shared_ptr< ThrustDirectionSettings >
         thrustDirectionGuidanceSettings,
 		const std::shared_ptr< ThrustMagnitudeSettings > thrustMagnitudeSettings )
 {
 ...
 }
````

### Examples: TUDATPY

#### TUDATPY class
````c++
    py::class_<tss::ThrustAccelerationSettings,
            std::shared_ptr<tss::ThrustAccelerationSettings>,
            tss::AccelerationSettings>(m, "ThrustAccelerationSettings",
                                       get_docstring("ThrustAccelerationSettings").c_str())
````

#### TUDATPY regular function
````c++
    m.def("aerodynamic", &tss::aerodynamicAcceleration,
          get_docstring("aerodynamic").c_str());
````

#### TUDATPY overloaded function
```c++
    m.def("thrust_acceleration", py::overload_cast<const std::shared_ptr<tss::ThrustDirectionSettings>,
                  const std::shared_ptr<tss::ThrustMagnitudeSettings>>(&tss::thrustAcceleration),
          py::arg("thrust_direction_settings"),
          py::arg("thrust_magnitude_settings"),
          get_docstring("thrust_acceleration", 0).c_str());
```

## Linking within API
TODO

Have not yet found out how to do that. For now make sure to put all class / function names that you refer to
within descriptions in single quotation marks, such as e.g. `EphemerisSettings`


## Linking external resources via urls
TODO

Have not yet found out how to do that elegantly. For now I simply copied the url in.


## Modifying exposure code while setting up docstrings
While setting up the docstrings, it is your responsibility to work over the exposure of the code that is being
documented and make the necessary adjustments. This includes:

- breaking down the exposure to its lowest level, meaning that each module exposure function shall be defined in its own file
- if a class to be documented has a constructor exposed (`.def(py::init< >, .. )`), comment out the exposure;
- if the class has get / set methods exposed, replace them by exposing the cpp getter/setter methods as an attribute,
  e.g.:
  ````
  .def_property("use_long_double_states", 
                &tss::TabulatedEphemerisSettings::getUseLongDoubleStates,
                &tss::TabulatedEphemerisSettings::setUseLongDoubleStates);
  ````
- if the class has only a get method exposed, replace it by exposing the cpp getter method as a readonly attribute,
  e.g.
  ````
  .def_property_readonly("body_state_history",
                         &tss::TabulatedEphemerisSettings::getBodyStateHistory)
  ````
  

## Full template

### Enums

TODO

### Base class

Template Base Class (for environment model settings object) - template taken from EphemerisSettings, where:

- `<XXX>` = Ephemeris
- `<pyattribute_1>` = some attribute of class, which via the get/set cpp functions is exposed as a class property in python
- `<pytype_1>`  = type of `<pyattribute_1>`
- `<description_1>` = brief description of `<pyattribute_1>`
- `<pyattribute_2>` = ... 


````
classes:

- name: <XXX>Settings
  short_summary: "Base class for providing settings for <XXX> model."
  extended_summary: |
  Functional (base) class for settings of <XXX> models that require no information in addition to their type.
  <XXX> model classes requiring additional information must be created using an object derived from this class.

  attributes:  # [py]  note: attributes for python only!
   note that <attribute_1> is read/write because tudatpy has get/reset functions for <attribute_1> (documented under methods).
    - name: <attribute_1> # [py]
      type: <pytype_1> # [py]
      description: <description_1> # [py]
   note that <attribute_2> is readonly because tudatpy has only a get functions to access <attribute_2> (documented under methods).
    - name: <attribute_2> # [py]  readonly
      type: <pytype_2> # [py]
      description: <description_2> # [py]

    -  ...


    methods:
     list the constructor as first method - python constructor docstrings no longer needed, since constructor will be removed from exposure.

     note that no parameters are given to discourage direct use of constructor (see **** classes and factory functions ****)
      - name: ctor # [cpp]
        short_summary: "Constructor." # [cpp]
        extended_summary: "Instances of this class are typically not generated by the user. Settings objects for XXX models should be instantiated through the factory functions of a derived class." # [cpp]

     now get/set methods of cpp class:

      - name: get<attribute_1> # [cpp]
          short_summary: "Retrieve <attribute_1>." // if <cpptype_1> is bool use phrase: "Check whether ..." # [cpp]
          extended_summary: "Function to retrieve <attribute_1>." // if <cpptype_1> is bool use phrase: "Function to retrieve boolean that..." # [cpp]

           parameters: there are no parameters in getter functions

          returns: # [cpp]
            - type: <cpptype_1> # [cpp]
              description: <description_1> # [cpp]

      - name: reset<attribute_1> # [cpp]
          short_summary: "Reset <attribute_1>." // if <cpptype_1> is bool use phrase: "Set whether  ..." # [cpp]
          extended_summary: "Function to reset <attribute_1>." // if <cpptype_1> is bool use phrase: "Function to set boolean that denotes whether  ..." # [cpp]

          parameters: # [cpp]
            - name: <attribute_1> # [cpp]
              type: <cpptype_1> # [cpp]
              description: <description_1> # [cpp]

           returns: there is no return in setter functions


      - name: get<attribute_2> # [cpp]
          short_summary: "Retrieve <attribute_2>." //  if <cpptype_2> is bool use phrase: "Check whether ..." # [cpp]
          extended_summary: "Function to retrieve <attribute_2>." //  if <cpptype_2> is bool use phrase: "Function to retrieve boolean that..." # [cpp]
           parameters: there are no parameters in getter functions
          returns: # [cpp]
            - type: <cpptype_2> # [cpp]
              description: <description_2> # [cpp]
````

### Derived class

Template Derived Class (for environment model settings object) (which has dedicated FF) - template taken from DirectSpiceEphemerisSettings, where:
- `<XXY>` = DirectSpiceEphemeris
- `<XXX>` = Ephemeris
- `<xxy>` = directSpiceEphemerisSettings (FF)

Public attributes & methods from `<XXX>` base class do not have to be re-documented in derived class (are inherited in API).

````
- name: <XXY>Settings
  short_summary: "Class for defining settings of <XXY>."
  extended_summary: "`<XXX>Settings` derived class for ephemeris which <minimal description of derived class>."

   attributes:
     same scheme as documenting attributes from base class: python only, distinction read/write

    methods:  python constructor docstrings no longer needed, since constructor will be removed from exposure.
      - name: ctor # [cpp]
        short_summary: "Constructor." # [cpp]
        extended_summary: "Instances of the `<XXY>Settings` class should be created through the `<xxy>` factory function." # [cpp]

       other methods, documented in the same style as in base class
````


### Free functions

Template for factory function of Derived Class (for environment model settings object) - template taken from directSpiceEphemerisSettings, where:
- `<XXY>` = DirectSpiceEphemeris
- `<XXX>` = Ephemeris
- `<xxy>` = directSpiceEphemerisSettings (FF)

````
functions:

- name: < python_name of <xxy> > # [py]
- name: <xxy>  # [cpp]
  short_summary: "Factory function for creating < brief description of the function that the created object serves >."
  extended_summary: |
  Factory function for settings object, < brief description of the function that the created object serves >.
  < additional info / context, mostly found on tudat-space/API/... website >
  This function creates an instance of an `<XXX>Settings` derived `<XXY>Settings` object.

  parameters:
    - name: <pyparametername_1> # [py]
      type: <pytype_1> # [py]
       if parameters have a default value -->  type: <pytype_1>, default=<default_value> # [py]
    - name: <parametername_1> # [cpp]
      type: <type_1> # [cpp]
      description: <description_1>

    - name: <pyparametername_2> # [py]
      type: <pytype_2> # [py]
       if parameters have a default value -->  type: <pytype_2>, default=<default_value> # [py]
    - name: <parametername_2> # [cpp]
      type: <type_2> # [cpp]
      description: <description_2>

    - ...

  returns:
    - type: <XXY>Settings  # [py]   class name of instantiated object
      description: [WIP] # [py]     return descriptions are WIP to incorporate whishes from Dominic, which I do not understand yet.
    - type: <XXY>Settings  # [cpp]  class name of instantiated object
      description: [WIP] # [cpp]    return descriptions are WIP to incorporate whishes from Dominic, which I do not understand yet.
````
