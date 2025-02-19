---
title: Migraciones - EF Core
author: bricelam
ms.author: bricelam
ms.date: 10/05/2018
uid: core/managing-schemas/migrations/index
ms.openlocfilehash: b94ac567644a9d98a05a40857cc072c500203370
ms.sourcegitcommit: 8f801993c9b8cd8a8fbfa7134818a8edca79e31a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2019
ms.locfileid: "59562564"
---
<a name="migrations"></a>Migraciones
==========

Un modelo de datos cambia durante el desarrollo y deja de estar sincronizado con la base de datos. Puede quitar la base de datos y dejar que EF cree una que coincida con el modelo, pero este procedimiento provoca la pérdida de datos. La característica de migraciones de EF Core proporciona una manera de actualizar incrementalmente el esquema de la base de datos para mantenerla sincronizada con el modelo de datos de la aplicación al tiempo que se conservan los datos existentes en la base de datos.

Las migraciones incluyen herramientas de línea de comandos y API que facilitan las siguientes tareas:

* [Crear una migración](#create-a-migration). Generar código que puede actualizar la base de datos para sincronizarla con un conjunto de cambios en el modelo.
* [Actualizar la base de datos](#update-the-database). Aplicar las migraciones pendientes para actualizar el esquema de la base de datos.
* [Personalizar el código de migración](#customize-migration-code). A veces el código generado debe modificarse o complementarse.
* [Quitar una migración](#remove-a-migration). Eliminar el código generado.
* [Revertir una migración](#revert-a-migration). Deshacer los cambios de la base de datos.
* [Generar scripts SQL](#generate-sql-scripts). Puede que necesite un script para actualizar una base de datos de producción o para solucionar problemas con el código de migración.
* [Aplicar migraciones en tiempo de ejecución](#apply-migrations-at-runtime). Si las actualizaciones en tiempo de diseño y la ejecución de scripts no son las mejores opciones, llame al método `Migrate()`.

<a name="install-the-tools"></a>Instalar las herramientas
-----------------

Instale las [herramientas de línea de comandos](xref:core/miscellaneous/cli/index):
* Para Visual Studio, se recomiendan las [herramientas de la Consola del Administrador de paquetes](xref:core/miscellaneous/cli/powershell).
* Para otros entornos de desarrollo, elija las [herramientas de la CLI de .NET Core](xref:core/miscellaneous/cli/dotnet).

<a name="create-a-migration"></a>Crear una migración
------------------

Una vez [definido el modelo inicial](xref:core/modeling/index), es el momento de crear la base de datos. Para agregar una migración inicial, ejecute el siguiente comando.

``` powershell
Add-Migration InitialCreate
```
``` Console
dotnet ef migrations add InitialCreate
```

Se agregan tres archivos al proyecto en el directorio **Migraciones**:

* **XXXXXXXXXXXXXX_InitialCreate.cs**: archivo principal de las migraciones. Contiene las operaciones necesarias para aplicar la migración (en `Up()`) y para revertirla (en `Down()`).
* **XXXXXXXXXXXXXX_InitialCreate.Designer.cs**: archivo de metadatos de las migraciones. Contiene información que usa EF.
* **MyContextModelSnapshot.cs**: instantánea del modelo actual. Se usa para determinar qué ha cambiado al agregar la siguiente migración.

La marca de tiempo del nombre del archivo ayuda a mantenerlos ordenados cronológicamente para que se pueda ver la progresión de cambios.

> [!TIP]
> Puede mover los archivos de Migraciones y cambiar su espacio de nombres. Se crean nuevas migraciones como elementos del mismo nivel de la última migración.

<a name="update-the-database"></a>Actualizar la base de datos
-------------------

Luego aplique la migración a la base de datos para crear el esquema.

``` powershell
Update-Database
```
``` Console
dotnet ef database update
```

<a name="customize-migration-code"></a>Personalizar el código de migración
------------------------

Después de realizar cambios en el modelo de EF Core, puede que el esquema de la base de datos no esté sincronizado. Para ponerlo al día, agregue otra migración. El nombre de la migración se puede usar como mensaje de confirmación en un sistema de control de versiones. Por ejemplo, podría elegir un nombre como *AddProductReviews* si el cambio es una nueva clase de entidad para las revisiones.

``` powershell
Add-Migration AddProductReviews
```
``` Console
dotnet ef migrations add AddProductReviews
```

Tras aplicar scaffolding a la migración (código generado para ella), revise el código para mayor precisión y agregue, quite o modifique todas las operaciones necesarias para aplicarla correctamente.

Por ejemplo, una migración podría contener las siguientes operaciones:

``` csharp
migrationBuilder.DropColumn(
    name: "FirstName",
    table: "Customer");

migrationBuilder.DropColumn(
    name: "LastName",
    table: "Customer");

migrationBuilder.AddColumn<string>(
    name: "Name",
    table: "Customer",
    nullable: true);
```

Aunque estas operaciones hacen que el esquema de la base de datos sea compatible, no conservan los nombres de cliente existentes. Para que sea mejor, vuelva a escribirla como se indica a continuación.

``` csharp
migrationBuilder.AddColumn<string>(
    name: "Name",
    table: "Customer",
    nullable: true);

migrationBuilder.Sql(
@"
    UPDATE Customer
    SET Name = FirstName + ' ' + LastName;
");

migrationBuilder.DropColumn(
    name: "FirstName",
    table: "Customer");

migrationBuilder.DropColumn(
    name: "LastName",
    table: "Customer");
```

> [!TIP]
> El proceso de scaffolding de la migración advierte si una operación puede ocasionar una pérdida de datos (como el borrado de una columna). Si aparece dicha advertencia, asegúrese especialmente de revisar el código de las migraciones para mayor precisión.

Aplique la migración a la base de datos con el comando apropiado.

``` powershell
Update-Database
```
``` Console
dotnet ef database update
```

### <a name="empty-migrations"></a>Migraciones vacías

A veces resulta útil agregar una migración sin realizar ningún cambio de modelo. En este caso, agregar una nueva migración crea archivos de código con clases vacías. Puede personalizar esta migración para llevar a cabo operaciones que no estén directamente relacionadas con el modelo de EF Core. Algunos aspectos que podría querer administrar de esta manera son:

* Búsqueda de texto completo
* Funciones
* Procedimientos almacenados
* Desencadenadores
* Vistas

<a name="remove-a-migration"></a>Quitar una migración
------------------
A veces uno agrega una migración y se da cuenta de que debe realizar cambios adicionales en el modelo de EF Core antes de aplicarla. Para quitar la última migración, use este comando.

``` powershell
Remove-Migration
```
``` Console
dotnet ef migrations remove
```

Después de quitar la migración, puede realizar los cambios de modelo adicionales y volver a agregarla.

<a name="revert-a-migration"></a>Revertir una migración
------------------
Si ya ha aplicado una migración (o varias migraciones) a la base de datos pero necesita revertirla, puede usar el mismo comando que para aplicar migraciones, aunque debe especificar el nombre de la migración que quiere revertir.

``` powershell
Update-Database LastGoodMigration
```
``` Console
dotnet ef database update LastGoodMigration
```

<a name="generate-sql-scripts"></a>Generar scripts SQL
--------------------
Al depurar las migraciones o implementarlas en una base de datos de producción, es útil generar un script SQL. El script luego se puede revisar y ajustar a las necesidades de una base de datos de producción. El script también puede usarse junto con una tecnología de implementación. El comando básico es el siguiente.

``` powershell
Script-Migration
```
``` Console
dotnet ef migrations script
```

Hay varias opciones para este comando.

La migración **from** debe ser la última migración aplicada a la base de datos antes de ejecutar el script. Si no se han aplicado migraciones, especifique `0` (es el valor predeterminado).

La migración **to** debe ser la última migración que se va a aplicar a la base de datos después de ejecutar el script. El valor predeterminado es la última migración del proyecto.

Se puede generar un script **idempotent** de forma opcional. Este script solo aplica migraciones si aún no se han aplicado a la base de datos. Es útil si no sabe exactamente cuál ha sido la última migración aplicada a la base de datos o si va a implementar en varias bases de datos que pueden encontrarse en migraciones diferentes.

<a name="apply-migrations-at-runtime"></a>Aplicar migraciones en tiempo de ejecución
---------------------------
Algunas aplicaciones pueden querer aplicar migraciones en tiempo de ejecución durante el inicio o la primera ejecución. Para ello, se usa el método `Migrate()`.

Este método se basa en el servicio `IMigrator`, que se puede usar para escenarios más avanzados. Use `DbContext.GetService<IMigrator>()` para acceder a él.

``` csharp
myDbContext.Database.Migrate();
```

> [!WARNING]
> * Este método no es para todos. Aunque es excelente para las aplicaciones con una base de datos local, la mayoría de las aplicaciones necesitan estrategias de implementación más sólidas, como la generación de scripts SQL.
> * No llame a `EnsureCreated()` antes de `Migrate()`. `EnsureCreated()` omite las migraciones para crear el esquema, lo cual provoca un error de `Migrate()`.

<a name="next-steps"></a>Pasos siguientes
----------

Para obtener más información, vea <xref:core/miscellaneous/cli/index>.
