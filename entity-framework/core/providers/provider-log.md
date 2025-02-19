---
title: Registro de cambios en el impacto de proveedor - EF Core
author: ajcvickers
ms.author: avickers
ms.date: 08/08/2018
ms.assetid: 7CEF496E-A5B0-4F5F-B68E-529609B23EF9
ms.technology: entity-framework-core
uid: core/providers/provider-log
ms.openlocfilehash: 61a58bd6119763d90731fac62343b983af510cb6
ms.sourcegitcommit: 87fcaba46535aa351db4bdb1231bd14b40e459b9
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2019
ms.locfileid: "59929880"
---
# <a name="provider-impacting-changes"></a>Cambios en el impacto de proveedor

Esta página contiene vínculos para extraer las solicitudes realizadas en el repositorio de EF Core que puede exigir que los autores de otros proveedores de base de datos para reaccionar ante ellos. La intención es proporcionar un punto de partida para los autores de proveedores de base de datos de terceros existentes al actualizar su proveedor para una nueva versión.

Este registro estamos empezando con los cambios de 2.1 a 2.2. Antes de 2.1 se utilizaba el [ `providers-beware` ](https://github.com/aspnet/EntityFrameworkCore/labels/providers-beware) y [ `providers-fyi` ](https://github.com/aspnet/EntityFrameworkCore/labels/providers-fyi) etiquetas en nuestros problemas y solicitudes de extracción.

## <a name="22-----30"></a>2.2 ---> 3.0

Tenga en cuenta que muchos de los [cambios importantes en el nivel de aplicación](../what-is-new/ef-core-3.0/breaking-changes.md) también afectará a los proveedores.

* https://github.com/aspnet/EntityFrameworkCore/pull/14022
  * Quitado API obsoletas y sobrecargas de parámetro opcional contraído
  * Removed DatabaseColumn.GetUnderlyingStoreType()
* https://github.com/aspnet/EntityFrameworkCore/pull/14589
  * Quitar API obsoletas
* https://github.com/aspnet/EntityFrameworkCore/pull/15044
  * Las subclases de CharTypeMapping pueden haber sido interrumpidas debido a cambios de comportamiento necesarios para corregir un par de errores en la implementación base.
* https://github.com/aspnet/EntityFrameworkCore/pull/15090
  * Agrega una clase base para IDatabaseModelFactory y actualizado para usar un objeto de parámetro para mitigar futuras interrupciones.
* https://github.com/aspnet/EntityFrameworkCore/pull/15123
  * Usar objetos de parámetro en MigrationsSqlGenerator para mitigar futuras interrupciones.
* https://github.com/aspnet/EntityFrameworkCore/pull/14972
  * Una configuración explícita de los niveles de registro que exige algunos cambios a las API que pueden estar usando los proveedores. En concreto, si los proveedores usan la infraestructura de registro directamente, este cambio puede interrumpir ese uso. Además, los proveedores que usan la infraestructura (que será pública) a partir de ahora deberá derivar de `LoggingDefinitions` o `RelationalLoggingDefinitions`. Consulte los proveedores de en memoria para ver ejemplos y SQL Server.
* https://github.com/aspnet/EntityFrameworkCore/pull/15091
  * Las cadenas de recursos principales, datos relacionales y abstracciones ahora son públicas.
  * `CoreLoggerExtensions` y `RelationalLoggerExtensions` ahora son públicos. Los proveedores deberían usar estas API al registro de eventos que se definen en el nivel relacional o un núcleo. No tienen acceso a recursos de registro directamente. Estos son todavía internos.
  * `IRawSqlCommandBuilder` ha cambiado de un servicio de singleton a un servicio con ámbito
  * `IMigrationsSqlGenerator` ha cambiado de un servicio de singleton a un servicio con ámbito
* https://github.com/aspnet/EntityFrameworkCore/pull/14706
  * La infraestructura para la creación de comandos relacionales se ha hecho pública para que pueda utilizado por los proveedores de forma segura y refactorizar ligeramente.
* https://github.com/aspnet/EntityFrameworkCore/pull/14733
  * `ILazyLoader` ha cambiado de un servicio con ámbito a un servicio transitorio
* https://github.com/aspnet/EntityFrameworkCore/pull/14610
  * `IUpdateSqlGenerator` ha cambiado de un servicio con ámbito a un servicio de singleton
  * Además, `ISingletonUpdateSqlGenerator` se ha quitado
* https://github.com/aspnet/EntityFrameworkCore/pull/15067
  * Mucho código interno que estaba siendo usada por los proveedores ya ha hecho público
  * Ya no sería necesario para hacer referencia a `IndentedStringBuilder` desde que se ha factorizado fuera de los lugares que lo exponen
  * Usos de `NonCapturingLazyInitializer` debe reemplazarse por `LazyInitializer` de la BCL
* https://github.com/aspnet/EntityFrameworkCore/pull/14608
  * Este cambio se trata íntegramente en el documento de cambios importantes de aplicación. Para los proveedores, esto puede afectar a más porque a menudo puede producir surge este problema, por lo que ha cambiado la infraestructura de prueba para que eso menos probable que las pruebas de EF core.
* https://github.com/aspnet/EntityFrameworkCore/issues/13961
  * `EntityMaterializerSource` se ha simplificado
* https://github.com/aspnet/EntityFrameworkCore/pull/14895
  * StartsWith traducción ha cambiado de forma que los proveedores que desea o necesita para reaccionar ante ellos
* https://github.com/aspnet/EntityFrameworkCore/pull/15168
  * Conjunto de servicios de convención ha cambiado. Los proveedores deben heredar "ProviderConventionSet" o "RelationalConventionSet".
  * Se pueden agregar personalizaciones a través de `IConventionSetCustomizer` servicios, pero esto está pensado para usarse por otras extensiones, no los proveedores.
  * Convenciones que se usan en tiempo de ejecución deben resolverse desde `IConventionSetBuilder`.
* [https://github.com/aspnet/EntityFrameworkCore/pull/15288](https://github.com/aspnet/EntityFrameworkCore/pull/15288) -Propagación de datos se ha refactorizado en una API pública para evitar la necesidad de utilizar los tipos internos. Solo esto debería afectar a los proveedores no relacionales, puesto que la propagación se controla mediante la clase base relacional para todos los proveedores relacionales.

## <a name="21-----22"></a>2.1 ---> 2.2

### <a name="test-only-changes"></a>Cambios sólo para pruebas

* [https://github.com/aspnet/EntityFrameworkCore/pull/12057](https://github.com/aspnet/EntityFrameworkCore/pull/12057) -Permitir delimitadores personalizable de SQL en las pruebas
  * Probar los cambios que permiten las comparaciones de punto flotante no estricta en BuiltInDataTypesTestBase
  * Cambios de prueba que permiten las pruebas de consulta pueden volver a usar con diferentes delimitadores SQL
* [https://github.com/aspnet/EntityFrameworkCore/pull/12072](https://github.com/aspnet/EntityFrameworkCore/pull/12072) -Agregue DbFunction pruebas a las pruebas de la especificación relacional
  * Por ejemplo, que se pueden ejecutar estas pruebas en todos los proveedores de base de datos
* [https://github.com/aspnet/EntityFrameworkCore/pull/12362](https://github.com/aspnet/EntityFrameworkCore/pull/12362) -Limpieza de la prueba Async
  * Quitar `Wait` , las llamadas innecesarias async y cambiar el nombre de algunos métodos de prueba
* [https://github.com/aspnet/EntityFrameworkCore/pull/12666](https://github.com/aspnet/EntityFrameworkCore/pull/12666) -Unificar la infraestructura de registro de prueba
  * Agregar `CreateListLoggerFactory` y quitado algunos infraestructura de registro anteriores, lo que requiere con estas pruebas para reaccionar ante los proveedores
* [https://github.com/aspnet/EntityFrameworkCore/pull/12500](https://github.com/aspnet/EntityFrameworkCore/pull/12500) -Ejecutar más pruebas de consulta tanto de forma sincrónica y asincrónica
  * Los nombres de prueba y la factorización ha cambiado, lo que requerirá con estas pruebas para reaccionar ante los proveedores
* [https://github.com/aspnet/EntityFrameworkCore/pull/12766](https://github.com/aspnet/EntityFrameworkCore/pull/12766) -Cambiar el nombre durante la navegación en el modelo ComplexNavigations
  * Proveedores de uso de estas pruebas que deba reaccionar
* [https://github.com/aspnet/EntityFrameworkCore/pull/12141](https://github.com/aspnet/EntityFrameworkCore/pull/12141) : Devuelve el contexto para el grupo en lugar de eliminarlos en las pruebas funcionales
  * Este cambio incluye algunas pruebas de refactorización que quizá requiera proveedores reaccionar ante ellos

### <a name="test-and-product-code-changes"></a>Cambios de código de prueba y producto

* [https://github.com/aspnet/EntityFrameworkCore/pull/12109](https://github.com/aspnet/EntityFrameworkCore/pull/12109) -Consolidar RelationalTypeMapping.Clone métodos
  * Se permiten cambios en el RelationalTypeMapping 2.1 para una simplificación de las clases derivadas. No creemos que esto era importantes en los proveedores, pero los proveedores pueden aprovechar las ventajas de este cambio en su tipo derivado asignar clases.
* [https://github.com/aspnet/EntityFrameworkCore/pull/12069](https://github.com/aspnet/EntityFrameworkCore/pull/12069) -Consultas con nombre o etiquetadas
  * Agrega la infraestructura para el etiquetado de las consultas LINQ y tener esas etiquetas se muestran como comentarios en el código SQL. Esto puede requerir que los proveedores reaccionar ante la generación de SQL.
* [https://github.com/aspnet/EntityFrameworkCore/pull/13115](https://github.com/aspnet/EntityFrameworkCore/pull/13115) -Compatibilidad con datos espaciales a través de interrupción
  * Permite que las asignaciones de tipos y miembros traductores registrarse fuera del proveedor
    * Los proveedores deben llamar a bases. FindMapping() en su implementación ITypeMappingSource para que funcione
  * Siga este patrón para agregar compatibilidad espacial en el proveedor que es coherente entre todos los proveedores.
* [https://github.com/aspnet/EntityFrameworkCore/pull/13199](https://github.com/aspnet/EntityFrameworkCore/pull/13199) -Agregar depuración mejorada para la creación del proveedor de servicio
  * Permite DbContextOptionsExtensions implementar una nueva interfaz que puede ayudarle a entender por qué se recompile el proveedor de servicios interno
* [https://github.com/aspnet/EntityFrameworkCore/pull/13289](https://github.com/aspnet/EntityFrameworkCore/pull/13289) -Agrega CanConnect API para su uso por las comprobaciones de mantenimiento
  * Esta solicitud agrega el concepto de `CanConnect` que se usará en el estado de ASP.NET Core comprueba para determinar si la base de datos está disponible. De forma predeterminada, la implementación relacional simplemente llama a `Exist`, pero los proveedores pueden implementar algo diferente si es necesario. No relacionales proveedores debe implementar la nueva API para la comprobación de estado que se pueda usar.
* [https://github.com/aspnet/EntityFrameworkCore/pull/13306](https://github.com/aspnet/EntityFrameworkCore/pull/13306) -Actualizar RelationalTypeMapping base para no establecer tamaño DbParameter
  * Detener configuración tamaño de forma predeterminada, ya que puede provocar el truncamiento. Los proveedores que necesite agregar su propia lógica si se debe establecer tamaño.
* (https://github.com/aspnet/EntityFrameworkCore/pull/13372) -RevEng: Especifique siempre el tipo de columna para columnas decimales
  * Configure siempre el tipo de columna para columnas decimales en código con scaffold en lugar de configurar por convención.
  * Los proveedores no deberían requerir cambios en su extremo.
* [https://github.com/aspnet/EntityFrameworkCore/pull/13469](https://github.com/aspnet/EntityFrameworkCore/pull/13469) -Agrega CaseExpression para generar las expresiones CASE de SQL
* [https://github.com/aspnet/EntityFrameworkCore/pull/13648](https://github.com/aspnet/EntityFrameworkCore/pull/13648) : Agrega la capacidad de especificar asignaciones de tipos en SqlFunctionExpression para mejorar el almacén de inferencia de tipos de argumentos y los resultados.
