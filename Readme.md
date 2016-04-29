# Test driver for running BaseX XQuery based test projects

[![Apache License 2.0](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](http://www.apache.org/licenses/LICENSE-2.0.html)
[![Latest version](http://img.shields.io/badge/latest%20version-1.0.2-blue.svg)](http://services.interactive-instruments.de/etfdev-af/release/de/interactive_instruments/etf/bsxm/etf-gmlgeox/1.0.2/etf-gmlgeox-1.0.2.jar)

The library can be used by the [ETF BaseX test driver](https://github.com/interactive-instruments/etf-bsxtd) to validate GML geometries within XML documents.

Please use the [etf-webapp project](https://github.com/interactive-instruments/etf-webapp) for
reporting [issues](https://github.com/interactive-instruments/etf-webapp/issues) or
[further documentation](https://github.com/interactive-instruments/etf-webapp/wiki).

The project can be build and installed by running the gradlew.sh/.bat wrapper with:
```gradle
$ gradlew build install
```

ETF component version numbers comply with the [Semantic Versioning Specification 2.0.0](http://semver.org/spec/v2.0.0.html).

ETF is an open source test framework developed by [interactive instruments](http://www.interactive-instruments.de/en) for testing geo network services and data.

## Installation in the etf-webapp
Currently there are external libraries that can not be initialized from within the package ClassLoader. Therefore you have to extract the content of the jar archive (jar xf etf-gmlgeox-x.jar) and copy the libraries from the root to the lib sub folder of your Basex testdriver _$driver/bsx/lib_ where the $driver directory is configured in your _etf-config.properties_ configuration file as variable _etf.testdrivers.dir_.

## Updating
Update the test driver and repeat the installation step.

## Geometry Support
The implementation of the module depends to a large extent on the deegree framework. The default geometry implementation of deegree does not support parsing all GML types, primarily the GML 3.3 types. Also, in a number of cases spatial operations are not supported for parsed geometries. This is primarily due to the fact that deegree relies on JTS to perform these operations, and that therefore the geometries must be simplified/linearized - which is not implemented for all geometry types that can be parsed by deegree. There is even a case where an incomplete JTS representations of a deegree geometry is accepted and used for spatial operations (for a surface with multiple polygon patches only the first is used)!

NOTE: Within this module, GML geometry elements are parsed by deegree, and then converted to JTS geometries via method `toJTSGeometry(Geometry)`in `GmlGeoXUtils.java` (at least for the spatial relationship operators).

Long story short, what this tells us is that a developer of this module must be very careful regarding which geometry types are supported for processing, and whether the computation is robust and exact or not. The documentation of a module function in both the source code itself (via Javadoc) and the test project developer manual should define the restrictions imposed by the implementation of the function.

For the spatial relationship operators, [this page](https://github.com/interactive-instruments/etf-webapp/wiki/gmlgeox-module-geometry-types-supported-by-spatial-operators) in the test project developer manual documents which geometry types are supported (also if they would be simplified/linearized) and which are not.

## Geometry Validation

Validation of GML geometry elements within a given XML node is basically a SAX-based scan for recognized GML geometry elements, and subsequent validation of these elements. The default set of recognized element names is a subset of GML. Functions offered by the module can be used to modify this set within an XQuery. See the [test project developer documentation for this module](https://github.com/interactive-instruments/etf-webapp/wiki/dev_manual_modules_gmlgeox) for further details.

## Deterministic vs. Non-Deterministic Functions

When exposing a Java method as an XQuery function through a module, BaseX offers a way to indicate if the according function is deterministic. By default, such a function is assumed to be non-deterministic. Deterministic functions allow optimization, i.e. caching results instead of re-evaluating a function each time it occurs in the query execution.

As stated on the [BaseX Wiki](http://docs.basex.org/wiki/Java_Bindings), **only** indicate that a Java method is deterministic if you know that it will have no side-effects and will always yield the same result. For the spatial relationship operators, for example, this is the case.
