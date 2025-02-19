---
title: 'Cambios importantes en EF Core 3.0: EF Core'
author: divega
ms.date: 02/19/2019
ms.assetid: EE2878C9-71F9-4FA5-9BC4-60517C7C9830
uid: core/what-is-new/ef-core-3.0/breaking-changes
ms.openlocfilehash: 9112d8d235237e68232aac54453d584af0edb524
ms.sourcegitcommit: b188194a1901f4d086d05765cbc5c9b8c9dc5eed
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2019
ms.locfileid: "66829492"
---
# <a name="breaking-changes-included-in-ef-core-30-currently-in-preview"></a>Cambios importantes incluidos en EF Core 3.0 (actualmente en versión preliminar)

> [!IMPORTANT]
> Tenga en cuenta que los conjuntos de características y las programaciones de las versiones futuras siempre están sujetos a cambios y, aunque intentamos mantener esta página actualizada, es posible que no refleje nuestros planes más recientes en todo momento.

Es posible que los siguientes cambios de API y comportamiento interrumpan las aplicaciones desarrolladas para EF Core 2.2. x cuando se actualicen a la versión 3.0.0.
Los cambios que esperamos que solo afecten a proveedores de base de datos se documentan en [Cambios para proveedores](../../providers/provider-log.md).
Aquí no se documentan los cambios en las características nuevas presentadas de una versión preliminar 3.0 a otra.

## <a name="linq-queries-are-no-longer-evaluated-on-the-client"></a>Las consultas LINQ ya no se evalúan en el cliente

[Problema de seguimiento n.° 14935](https://github.com/aspnet/EntityFrameworkCore/issues/14935)
[Consulte también el problema n.° 12795](https://github.com/aspnet/EntityFrameworkCore/issues/12795)

Este cambio se ha introducido en EF Core 3.0 (versión preliminar 4).

**Comportamiento anterior**

Antes de 3.0, cuando en EF Core no se podía convertir una expresión que formaba parte de una consulta SQL o un parámetro, la expresión se evaluaba de forma automática en el cliente.
De forma predeterminada, la evaluación de cliente de expresiones potencialmente costosas solo desencadenaba una advertencia.

**Comportamiento nuevo**

A partir de 3.0, en EF Core solo se permite que se evalúen en el cliente las expresiones en la proyección de nivel superior (la última llamada a `Select()` de la consulta).
Cuando las expresiones de otra parte de la consulta no se pueden convertir en SQL o un parámetro, se inicia una excepción.

**Por qué**

La evaluación de cliente automática de las consultas permite que se ejecuten muchas consultas incluso si no se pueden convertir elementos importantes de ellas.
Esto puede provocar un comportamiento inesperado y potencialmente dañino que es posible que solo sea evidente en entornos de producción.
Por ejemplo, una condición en una llamada a `Where()` que no se puede convertir puede provocar que todas las filas de la tabla se transfieran desde el servidor de base de datos y que el filtro se aplique en el cliente.
Esta situación puede pasar desapercibida fácilmente si la tabla solo contiene algunas filas en la fase de desarrollo, pero ser más grave cuando la aplicación pase a producción, donde la tabla puede contener millones de filas.
Las advertencias de evaluación de cliente también se suelen pasar por alto durante el desarrollo.

Además de esto, la evaluación de cliente automática puede causar problemas en los que la mejora de la traducción de consultas para expresiones específicas provocaba cambios importantes no deseados entre versiones.

**Mitigaciones**

Si una consulta no se puede traducir totalmente, vuelva a escribirla en un formato que se pueda traducir, o bien use `AsEnumerable()`, `ToList()` o una función similar para devolver los datos al cliente de forma explícita, donde después se puedan seguir procesando mediante LINQ to Objects.

## <a name="entity-framework-core-is-no-longer-part-of-the-aspnet-core-shared-framework"></a>Entity Framework Core ya no forma parte del marco compartido ASP.NET Core

[Anuncios del problema de seguimiento n.º 325](https://github.com/aspnet/Announcements/issues/325)

Este cambio se ha introducido en ASP.NET Core 3.0 (versión preliminar 1). 

**Comportamiento anterior**

Antes de ASP.NET Core 3.0, cuando se agregaba una referencia de paquete a `Microsoft.AspNetCore.App` o `Microsoft.AspNetCore.All`, se incluía EF Core y algunos de los proveedores de datos de EF Core, como el de SQL Server.

**Comportamiento nuevo**

A partir de la versión 3.0, el marco compartido ASP.NET Core no incluye EF Core ni ningún proveedor de datos de EF Core.

**Por qué**

Antes de este cambio, para obtener EF Core se necesitaban varios pasos en función de si la aplicación se destinaba a ASP.NET Core y SQL Server o no. Además, la actualización de ASP.NET Core forzaba la de EF Core y el proveedor de SQL Server, lo que no siempre es deseable.

Con este cambio, la experiencia de obtención de EF Core es la misma en todos los proveedores, implementaciones admitidas de .NET y tipos de aplicación.
Ahora los desarrolladores también pueden controlar exactamente cuándo se actualizan EF Core y los proveedores de datos de EF Core.

**Mitigaciones**

Para usar EF Core en una aplicación ASP.NET Core 3.0 o cualquier otra aplicación compatible, debe agregar de forma explícita una referencia de paquete al proveedor de base de datos de EF Core que se va a usar en la aplicación.

## <a name="the-ef-core-command-line-tool-dotnet-ef-is-no-longer-part-of-the-net-core-sdk"></a>La herramienta de línea de comandos de EF Core, dotnet ef, ya no forma parte del SDK de .NET Core

[Problema de seguimiento n.º 14016](https://github.com/aspnet/EntityFrameworkCore/issues/14016)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 4) y la versión correspondiente del SDK de .NET Core.

**Comportamiento anterior**

Antes de 3.0, la herramienta `dotnet ef` se incluía en el SDK de .NET Core y estaba disponible para usarse desde la línea de comandos de cualquier proyecto sin necesidad de realizar pasos adicionales. 

**Comportamiento nuevo**

A partir de la versión 3.0, el SDK de .NET no incluye la herramienta `dotnet ef`, por lo que antes de poder usarla tendrá que instalarla de forma explícita como una herramienta local o global. 

**Por qué**

Este cambio nos permite distribuir y actualizar `dotnet ef` como una herramienta convencional de la CLI de .NET en NuGet, coherente con el hecho de que la versión 3.0 de EF Core también se distribuye siempre como un paquete NuGet.

**Mitigaciones**

Para poder administrar las migraciones o aplicar la técnica de scaffolding a `DbContext`, instale `dotnet-ef` con el comando `dotnet tool install`.
Por ejemplo, para instalarlo como una herramienta global, puede escribir este comando:

  ``` console
  $ dotnet tool install --global dotnet-ef --version <exact-version>
  ```

También se puede obtener una herramienta local cuando se restauran las dependencias de un proyecto que la declara como una dependencia de herramientas mediante un [archivo de manifiesto de herramientas](https://github.com/dotnet/cli/issues/10288).

## <a name="fromsql-executesql-and-executesqlasync-have-been-renamed"></a>FromSql, ExecuteSql y ExecuteSqlAsync han cambiado de nombre

[Problema de seguimiento n.º 10996](https://github.com/aspnet/EntityFrameworkCore/issues/10996)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 4).

**Comportamiento anterior**

Antes de EF Core 3.0, estos nombres de métodos se sobrecargaban para funcionar tanto con una cadena normal como con una cadena que se debería interpolar en SQL y parámetros.

**Comportamiento nuevo**

A partir de la versión EF Core 3.0, use `FromSqlRaw`, `ExecuteSqlRaw` y `ExecuteSqlRawAsync` para crear una consulta con parámetros donde los parámetros se pasan por separado de la cadena de consulta.
Por ejemplo:

```C#
context.Products.FromSqlRaw(
    "SELECT * FROM Products WHERE Name = {0}",
    product.Name);
```

Use `FromSqlInterpolated`, `ExecuteSqlInterpolated` y `ExecuteSqlInterpolatedAsync` para crear una consulta con parámetros donde los parámetros se pasan como parte de una cadena de consulta interpolada.
Por ejemplo:

```C#
context.Products.FromSqlInterpolated(
    $"SELECT * FROM Products WHERE Name = {product.Name}");
```

Tenga en cuenta que las dos consultas anteriores producirán el mismo código SQL parametrizado con los mismos parámetros SQL.

**Por qué**

Las sobrecargas del método como esta facilitan las llamadas accidentales al método de cadena sin procesar cuando la intención era llamar al método de cadena interpolada y viceversa.
Esto podría resultar en consultas que no se parametrizan cuando deberían.

**Mitigaciones**

Haga el cambio para usar los nuevos nombres de métodos.

## <a name="fromsql-methods-can-only-be-specified-on-query-roots"></a>Solo se pueden especificar métodos de FromSql en raíces de consulta.

[Problema de seguimiento n.° 15704](https://github.com/aspnet/EntityFrameworkCore/issues/15704)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 6).

**Comportamiento anterior**

Antes de EF Core 3.0, el método `FromSql` podía especificarse en cualquier lugar en la consulta.

**Comportamiento nuevo**

A partir de EF Core 3.0, los nuevos métodos `FromSqlRaw` y `FromSqlInterpolated` (que reemplazan a `FromSql`) solo pueden especificarse en las raíces de la consulta, es decir, directamente en `DbSet<>`. Si intenta especificarlos en cualquier otro lugar se producirá un error de compilación.

**Por qué**

La especificación de `FromSql` en cualquier otro lugar diferente de `DbSet` no tenía un significado o valor agregado, y podría causar ambigüedad en ciertos escenarios.

**Mitigaciones**

Las invocaciones de `FromSql` se deben mover para que estén directamente en el `DbSet` al que se aplican.

## <a name="query-execution-is-logged-at-debug-level"></a>La ejecución de consultas se registra en el nivel de depuración

[Problema de seguimiento n.º 14523](https://github.com/aspnet/EntityFrameworkCore/issues/14523)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 3).

**Comportamiento anterior**

Antes de EF Core 3.0, la ejecución de consultas y otros comandos se registraba en el nivel `Info`.

**Comportamiento nuevo**

A partir de EF Core 3.0, el registro de ejecución de comandos y SQL se produce en el nivel `Debug`.

**Por qué**

Este cambio se ha realizado para reducir el ruido en el nivel de registro `Info`.

**Mitigaciones**

Este evento de registro se define mediante `RelationalEventId.CommandExecuting` con el id. de evento 20100.
Para volver a registrar SQL en el nivel `Info`, configure explícitamente el nivel en `OnConfiguring` o en `AddDbContext`.
Por ejemplo:
```C#
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    => optionsBuilder
        .UseSqlServer(connectionString)
        .ConfigureWarnings(c => c.Log((RelationalEventId.CommandExecuting, LogLevel.Info)));
```

## <a name="temporary-key-values-are-no-longer-set-onto-entity-instances"></a>Los valores de clave temporal ya no se establecen en instancias de entidad

[Problema de seguimiento n.º 12378](https://github.com/aspnet/EntityFrameworkCore/issues/12378)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 2).

**Comportamiento anterior**

Antes de EF Core 3.0, los valores temporales se asignaban a todas las propiedades de clave para las que posteriormente la base de datos generaba un valor real.
Normalmente, estos valores temporales eran números negativos grandes.

**Comportamiento nuevo**

A partir de la versión 3.0, en EF Core se almacena el valor de clave temporal como parte de la información de seguimiento de la entidad y la propiedad clave en sí no se modifica.

**Por qué**

Este cambio se ha realizado para evitar que los valores de clave temporales se conviertan erróneamente en permanentes cuando una entidad de la que previamente una instancia de `DbContext` ha realizado el seguimiento se mueve a otra instancia de `DbContext`. 

**Mitigaciones**

Las aplicaciones que asignan valores de clave principal a claves externas para crear asociaciones entre entidades pueden depender del comportamiento anterior si las claves principales son generadas por el almacén y pertenecen a entidades en el estado `Added`.
Esto se puede evitar con las siguientes situaciones:
* No se usan claves generadas por el almacén.
* Se establecen propiedades de navegación para crear relaciones en lugar de establecer valores de clave externa.
* Se obtienen los valores de clave temporal reales de la información de seguimiento de la entidad.
Por ejemplo, `context.Entry(blog).Property(e => e.Id).CurrentValue` devolverá el valor temporal aunque no se haya establecido `blog.Id`.

## <a name="detectchanges-honors-store-generated-key-values"></a>DetectChanges respeta los valores de clave generados por el almacén

[Problema de seguimiento n.º 14616](https://github.com/aspnet/EntityFrameworkCore/issues/14616)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 3).

**Comportamiento anterior**

Antes de EF Core 3.0, se realizaba el seguimiento en el estado `Added` de las entidades sin seguimiento detectadas por `DetectChanges` y se insertaban como una fila nueva cuando se llamaba a `SaveChanges`.

**Comportamiento nuevo**

A partir de EF Core 3.0, si una entidad usa valores de clave generados y se establece un valor de clave, se realizará el seguimiento de la entidad en el estado `Modified`.
Esto significa que se supone que existe una fila para la entidad y que se actualizará cuando se llame a `SaveChanges`.
Si no se establece el valor de clave, o bien si el tipo de entidad no usa claves generadas, se seguirá realizando el seguimiento de la entidad nueva como `Added` al igual que en las versiones anteriores.

**Por qué**

Este cambio se ha realizado para que sea más sencillo y coherente trabajar con gráficos de entidades desconectadas mientras se usan claves generadas por el almacén.

**Mitigaciones**

Este cambio puede interrumpir una aplicación si se configura un tipo de entidad para usar claves generadas, pero se establecen de forma explícita valores de clave para las instancias nuevas.
La solución consiste en configurar de forma explícita las propiedades de clave para que no usen valores generados.
Por ejemplo, con la API fluida:

```C#
modelBuilder
    .Entity<Blog>()
    .Property(e => e.Id)
    .ValueGeneratedNever();
```

O bien con anotaciones de datos:

```C#
[DatabaseGenerated(DatabaseGeneratedOption.None)]
public string Id { get; set; }
```

## <a name="cascade-deletions-now-happen-immediately-by-default"></a>Las eliminaciones en cascada ahora se realizan inmediatamente de forma predeterminada

[Problema de seguimiento n.º 10114](https://github.com/aspnet/EntityFrameworkCore/issues/10114)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 3).

**Comportamiento anterior**

Antes de la versión 3.0, en EF Core no se aplicaban acciones en cascada (eliminación de entidades dependientes cuando se eliminaba una entidad de seguridad obligatoria o cuando se rompía la relación con una entidad de seguridad obligatoria) hasta que se llamaba a SaveChanges.

**Comportamiento nuevo**

A partir de 3.0, en EF Core las acciones en cascada se aplican en cuanto se detecta la condición desencadenadora.
Por ejemplo, como resultado de la llamada a `context.Remove()` para eliminar una entidad de seguridad, todos los dependientes obligatorios relacionados de los que se realiza el seguimiento también se establecen en `Deleted` inmediatamente.

**Por qué**

Este cambio se ha realizado para mejorar la experiencia en escenarios de auditoría y enlace de datos donde es importante comprender qué entidades se van a eliminar _antes_ de llamar a `SaveChanges`.

**Mitigaciones**

El comportamiento anterior se puede restaurar mediante opciones de `context.ChangedTracker`.
Por ejemplo:

```C#
context.ChangeTracker.CascadeDeleteTiming = CascadeTiming.OnSaveChanges;
context.ChangeTracker.DeleteOrphansTiming = CascadeTiming.OnSaveChanges;
```

## <a name="deletebehaviorrestrict-has-cleaner-semantics"></a>DeleteBehavior.Restrict tiene una semántica más limpia

[Problema de seguimiento n.º 12661](https://github.com/aspnet/EntityFrameworkCore/issues/12661)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 5).

**Comportamiento anterior**

Antes de la versión 3.0, `DeleteBehavior.Restrict` creaba claves externas en la base de datos con la semántica `Restrict`, pero también realizaba una corrección interna de manera no evidente.

**Comportamiento nuevo**

A partir de la versión 3.0, `DeleteBehavior.Restrict` garantiza que las claves externas se crean con la semántica `Restrict`, es decir, sin cascadas y realizando una infracción de restricción, sin llevar a cabo correcciones internas de EF.

**Por qué**

Este cambio se realizó para mejorar la experiencia de uso de `DeleteBehavior` de manera intuitiva sin efectos secundarios inesperados.

**Mitigaciones**

El comportamiento anterior se puede restaurar con `DeleteBehavior.ClientNoAction`.

## <a name="query-types-are-consolidated-with-entity-types"></a>Los tipos de consulta se consolidan con tipos de entidad

[Problema de seguimiento n.º 14194](https://github.com/aspnet/EntityFrameworkCore/issues/14194)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 3).

**Comportamiento anterior**

Antes de EF Core 3.0, los [tipos de consulta](xref:core/modeling/query-types) eran un medio para consultar los datos que no definen una clave principal de una manera estructurada.
Es decir, un tipo de consulta se usaba para asignar tipos de entidad sin claves (más probablemente desde una vista, pero posiblemente desde una tabla), mientras que un tipo de entidad estándar se usaba cuando había una clave disponible (más probablemente desde una tabla, pero posiblemente desde una vista).

**Comportamiento nuevo**

Ahora un tipo de consulta se convierte en un tipo de entidad sin una clave principal.
Los tipos de entidad sin clave tienen la misma funcionalidad que los tipos de consulta de las versiones anteriores.

**Por qué**

Este cambio se ha realizado para reducir la confusión en torno a la finalidad de los tipos de consulta.
En concreto, son tipos de entidad sin clave y, por ello, intrínsecamente son de solo lectura, pero no se deben usar solo porque un tipo de entidad tenga que ser de solo lectura.
Del mismo modo, se suelen asignar a vistas, pero solo porque las vistas no suelen definir claves.

**Mitigaciones**

Los elementos siguientes de la API ahora están obsoletos:
* **`ModelBuilder.Query<>()`** : en su lugar es necesario llamar a `ModelBuilder.Entity<>().HasNoKey()` para marcar un tipo de entidad como sin claves.
Esto todavía no se configurará por convención para evitar una configuración incorrecta cuando se espera una clave principal, pero no coincide con la convención.
* **`DbQuery<>`** : en su lugar se debe usar `DbSet<>`.
* **`DbContext.Query<>()`** : en su lugar se debe usar `DbContext.Set<>()`.

## <a name="configuration-api-for-owned-type-relationships-has-changed"></a>La API de configuración para las relaciones de tipo de propiedad ha cambiado

[Problema de seguimiento n.º 12444](https://github.com/aspnet/EntityFrameworkCore/issues/12444)
[Problema de seguimiento n.º 9148](https://github.com/aspnet/EntityFrameworkCore/issues/9148)
[Problema de seguimiento n.º 14153](https://github.com/aspnet/EntityFrameworkCore/issues/14153)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 3).

**Comportamiento anterior**

Antes de EF Core 3.0, la configuración de la relación de propiedad se realizaba directamente después de la llamada a `OwnsOne` o `OwnsMany`. 

**Comportamiento nuevo**

A partir de EF Core 3.0, ahora hay una API fluida para configurar una propiedad de navegación para el propietario mediante `WithOwner()`.
Por ejemplo:

```C#
modelBuilder.Entity<Order>.OwnsOne(e => e.Details).WithOwner(e => e.Order);
```

La configuración relacionada con la relación entre el propietario y lo que se posee ahora se debe encadenar después de `WithOwner()`, de forma similar a cómo se configuran otras relaciones.
Pero la configuración del propio tipo de propiedad se seguirá encadenando después de `OwnsOne()/OwnsMany()`.
Por ejemplo:

```C#
modelBuilder.Entity<Order>.OwnsOne(e => e.Details, eb =>
    {
        eb.WithOwner()
            .HasForeignKey(e => e.AlternateId)
            .HasConstraintName("FK_OrderDetails");
            
        eb.ToTable("OrderDetails");
        eb.HasKey(e => e.AlternateId);
        eb.HasIndex(e => e.Id);

        eb.HasOne(e => e.Customer).WithOne();

        eb.HasData(
            new OrderDetails
            {
                AlternateId = 1,
                Id = -1
            });
    });
```

Además, la llamada a `Entity()`, `HasOne()` o `Set()` con un tipo de propiedad de destino ahora iniciará una excepción.

**Por qué**

Este cambio se ha realizado para crear una separación más clara entre la configuración del propio tipo de propiedad y la _relación con_ el tipo de propiedad.
A su vez, esto elimina la ambigüedad y la confusión de métodos como `HasForeignKey`.

**Mitigaciones**

Cambie la configuración de las relaciones de tipo de propiedad para usar la nueva superficie de API, como se muestra en el ejemplo anterior.

## <a name="dependent-entities-sharing-the-table-with-the-principal-are-now-optional"></a>Ahora, las entidades dependientes que comparten la tabla con la entidad de seguridad son opcionales

[Problema de seguimiento n.º 9005](https://github.com/aspnet/EntityFrameworkCore/issues/9005)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 4).

**Comportamiento anterior**

Considere el modelo siguiente:
```C#
public class Order
{
    public int Id { get; set; }
    public int CustomerId { get; set; }
    public OrderDetails Details { get; set; }
}

public class OrderDetails
{
    public int Id { get; set; }
    public string ShippingAddress { get; set; }
}
```
Antes de EF Core 3.0, si `OrderDetails` era propiedad de `Order` o estaba asignado explícitamente a la misma tabla, siempre era necesaria una instancia de `OrderDetails` al agregar un elemento `Order` nuevo.


**Comportamiento nuevo**

A partir de la versión 3.0, EF Core permite agregar `Order` sin `OrderDetails` y asigna todas las propiedades `OrderDetails` a excepción de la clave principal a columnas que aceptan valores NULL.
Al realizar consultas, EF Core establece `OrderDetails` en `null` si ninguna de las propiedades necesarias tiene un valor o si no tiene propiedades necesarias más allá de la clave principal y todas las propiedades son `null`.

**Mitigaciones**

Si el modelo tiene una tabla que comparte dependencias con todas las columnas opcionales, pero la navegación que apunta a ella no se espera que sea `null`, la aplicación debería modificarse para controlar los casos en los que la navegación sea `null`. Si esto no es posible, debería agregarse una propiedad necesaria al tipo de entidad o, al menos, una entidad debería tener un valor distinto a `null` asignado.

## <a name="all-entities-sharing-a-table-with-a-concurrency-token-column-have-to-map-it-to-a-property"></a>Todas las entidades que compartan una tabla con una columna de token de simultaneidad tienen que asignarla a una propiedad

[Problema de seguimiento n.º 14154](https://github.com/aspnet/EntityFrameworkCore/issues/14154)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 4).

**Comportamiento anterior**

Considere el modelo siguiente:
```C#
public class Order
{
    public int Id { get; set; }
    public int CustomerId { get; set; }
    public byte[] Version { get; set; }
    public OrderDetails Details { get; set; }
}

public class OrderDetails
{
    public int Id { get; set; }
    public string ShippingAddress { get; set; }
}

protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Order>()
        .Property(o => o.Version).IsRowVersion().HasColumnName("Version");
}
```
Antes de EF Core 3.0, si `OrderDetails` era propiedad de `Order` o estaba asignado explícitamente a la misma tabla, si solo se actualizaba `OrderDetails`, no se actualizaba el valor `Version` en el cliente y se producía un error en la próxima actualización.


**Comportamiento nuevo**

A partir de la versión 3.0, EF Core propaga el nuevo valor `Version` en `Order` si posee `OrderDetails`. En caso contrario, se produce una excepción durante la validación del modelo.

**Por qué**

Este cambio se realizó para evitar un valor de token de simultaneidad obsoleto cuando solo se actualiza una de las entidades asignadas a la misma tabla.

**Mitigaciones**

Todas las entidades que comparten la tabla deben incluir una propiedad que se asigna a la columna del token de simultaneidad. Es posible crear una en estado reemplazado:
```C#
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<OrderDetails>()
        .Property<byte[]>("Version").IsRowVersion().HasColumnName("Version");
}
```

## <a name="inherited-properties-from-unmapped-types-are-now-mapped-to-a-single-column-for-all-derived-types"></a>Ahora, las propiedades heredadas de tipos sin asignar se asignan a una única columna para todos los tipos derivados

[Problema de seguimiento n.º 13998](https://github.com/aspnet/EntityFrameworkCore/issues/13998)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 4).

**Comportamiento anterior**

Considere el modelo siguiente:
```C#
public abstract class EntityBase
{
    public int Id { get; set; }
}

public abstract class OrderBase : EntityBase
{
    public int ShippingAddress { get; set; }
}

public class BulkOrder : OrderBase
{
}

public class Order : OrderBase
{
}

protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Ignore<OrderBase>();
    modelBuilder.Entity<EntityBase>();
    modelBuilder.Entity<BulkOrder>();
    modelBuilder.Entity<Order>();
}
```

Antes de EF Core 3.0, la propiedad `ShippingAddress` se asignaba a columnas distintas para `BulkOrder` y `Order` de forma predeterminada.

**Comportamiento nuevo**

A partir de la versión3.0, EF Core solo crea una columna para `ShippingAddress`.

**Por qué**

El comportamiento anterior no era el esperado.

**Mitigaciones**

Todavía se puede asignar explícitamente la propiedad a columnas separadas en los tipos derivados:

```C#
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Ignore<OrderBase>();
    modelBuilder.Entity<EntityBase>();
    modelBuilder.Entity<BulkOrder>()
        .Property(o => o.ShippingAddress).HasColumnName("BulkShippingAddress");
    modelBuilder.Entity<Order>()
        .Property(o => o.ShippingAddress).HasColumnName("ShippingAddress");
}
```

## <a name="the-foreign-key-property-convention-no-longer-matches-same-name-as-the-principal-property"></a>La convención de propiedad de clave externa ya no coincide con el mismo nombre que la propiedad de entidad de seguridad

[Problema de seguimiento n.º 13274](https://github.com/aspnet/EntityFrameworkCore/issues/13274)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 3).

**Comportamiento anterior**

Considere el modelo siguiente:
```C#
public class Customer
{
    public int CustomerId { get; set; }
    public ICollection<Order> Orders { get; set; }
}

public class Order
{
    public int Id { get; set; }
    public int CustomerId { get; set; }
}
```
Antes de EF Core 3.0, se podía usar la propiedad `CustomerId` para la clave externa por convención.
Pero si `Order` es un tipo de propiedad, entonces esto convertiría también a `CustomerId` en la clave principal, algo que no suele ser lo esperado.

**Comportamiento nuevo**

A partir de la versión 3.0, EF Core no intenta usar las propiedades de claves externas por convención si tienen el mismo nombre que la propiedad de entidad de seguridad.
Los patrones de nombre de tipo de entidad de seguridad concatenado con el nombre de propiedad de la entidad de seguridad y de nombre de navegación concatenado con el nombre de propiedad de la entidad de seguridad todavía se hacen coincidir.
Por ejemplo:

```C#
public class Customer
{
    public int Id { get; set; }
    public ICollection<Order> Orders { get; set; }
}

public class Order
{
    public int Id { get; set; }
    public int CustomerId { get; set; }
}
```

```C#
public class Customer
{
    public int Id { get; set; }
    public ICollection<Order> Orders { get; set; }
}

public class Order
{
    public int Id { get; set; }
    public int BuyerId { get; set; }
    public Customer Buyer { get; set; }
}
```

**Por qué**

Este cambio se ha realizado para evitar definir erróneamente una propiedad de clave principal en el tipo de propiedad.

**Mitigaciones**

Si la propiedad se ha diseñado para ser la clave externa y, por tanto, parte de la clave principal, se debe configurar explícitamente como tal.

## <a name="database-connection-is-now-closed-if-not-used-anymore-before-the-transactionscope-has-been-completed"></a>Ahora, la conexión de base de datos se cierra si ya no se usa antes de que se complete TransactionScope

[Problema de seguimiento n.º 14218](https://github.com/aspnet/EntityFrameworkCore/issues/14218)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 4).

**Comportamiento anterior**

Antes de EF Core 3.0, si el contexto abría la conexión dentro de `TransactionScope`, la conexión permanecía abierta mientras el ámbito actual `TransactionScope` estuviese activo.

```C#
using (new TransactionScope())
{
    using (AdventureWorks context = new AdventureWorks())
    {
        context.ProductCategories.Add(new ProductCategory());
        context.SaveChanges();

        // Old behavior: Connection is still open at this point
        
        var categories = context.ProductCategories().ToList();
    }
}
```

**Comportamiento nuevo**

A partir de la versión 3.0, EF Core cierra la conexión en cuanto se deja de usar.

**Por qué**

Este cambio permite usar varios contextos en el mismo ámbito `TransactionScope`. El comportamiento nuevo también coincide con el de EF6.

**Mitigaciones**

Si la conexión debe permanecer abierta, una llamada explícita a `OpenConnection()` asegurará que EF Core no la cierre de forma prematura:

```C#
using (new TransactionScope())
{
    using (AdventureWorks context = new AdventureWorks())
    {
        context.Database.OpenConnection();
        context.ProductCategories.Add(new ProductCategory());
        context.SaveChanges();
        
        var categories = context.ProductCategories().ToList();
        context.Database.CloseConnection();
    }
}
```

## <a name="each-property-uses-independent-in-memory-integer-key-generation"></a>Cada propiedad usa la generación de claves enteras en memoria independiente

[Problema de seguimiento n.º 6872](https://github.com/aspnet/EntityFrameworkCore/issues/6872)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 4).

**Comportamiento anterior**

Antes de EF Core 3.0, se usaba un generador de valores compartidos para todas las propiedades de clave entera en memoria.

**Comportamiento nuevo**

A partir de EF Core 3.0, cada propiedad de clave entera obtiene su propio generador de valores cuando se usa la base de datos en memoria.
Además, si se elimina la base de datos, la generación de claves se restablece para todas las tablas.

**Por qué**

Este cambio se ha realizado para alinear la generación de claves en memoria más estrechamente a la generación de claves de base de datos reales y para mejorar la capacidad para aislar las pruebas entre sí cuando se usa la base de datos en memoria.

**Mitigaciones**

Esto puede interrumpir una aplicación que se base en el establecimiento de valores de clave específicos en memoria.
En su lugar, considere la posibilidad de no depender de valores de clave específicos, o bien de actualizar para que coincida con el comportamiento nuevo.

## <a name="backing-fields-are-used-by-default"></a>Los campos de respaldo se usan de forma predeterminada

[Problema de seguimiento n.º 12430](https://github.com/aspnet/EntityFrameworkCore/issues/12430)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 2).

**Comportamiento anterior**

Antes de la versión 3.0, incluso si se conocía el campo de respaldo de una propiedad, de forma predeterminada en EF Core se leía y escribía el valor de propiedad mediante los métodos captadores y establecedores de propiedades.
La excepción era la ejecución de consultas, donde el campo de respaldo se establecía directamente si se conocía.

**Comportamiento nuevo**

A partir de EF Core 3.0, si se conoce el campo de respaldo para una propiedad, EF Core siempre la leerá y escribirá mediante el campo de respaldo.
Esto podría provocar una interrupción de la aplicación si depende de un comportamiento adicional codificado en los métodos captadores o establecedores.

**Por qué**

Este cambio se ha realizado para evitar que EF Core desencadene erróneamente lógica de negocios de forma predeterminada al realizar operaciones de base de datos que implican entidades.

**Mitigaciones**

El comportamiento anterior a la versión 3.0 se puede restaurar mediante la configuración del modo de acceso de propiedad en `ModelBuilder`.
Por ejemplo:

```C#
modelBuilder.UsePropertyAccessMode(PropertyAccessMode.PreferFieldDuringConstruction);
```

## <a name="throw-if-multiple-compatible-backing-fields-are-found"></a>Inicio de excepciones si se encuentran varios campos de respaldo compatibles

[Problema de seguimiento n.º 12523](https://github.com/aspnet/EntityFrameworkCore/issues/12523)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 4).

**Comportamiento anterior**

Antes de EF Core 3.0, si varios campos coincidían con las reglas para buscar el campo de respaldo de una propiedad, se elegía un campo según un orden de prioridad.
Esto podía producir que, en caso de ambigüedad, se usara el campo incorrecto.

**Comportamiento nuevo**

A partir de EF Core 3.0, si varios campos coinciden con la misma propiedad, se inicia una excepción.

**Por qué**

Este cambio se ha realizado para evitar de forma silenciosa el uso de un campo con respecto a otro cuando solo uno puede ser correcto.

**Mitigaciones**

En las propiedades con campos de respaldo ambiguos se debe especificar de forma explícita el campo que se va usar.
Por ejemplo, con la API fluida:

```C#
modelBuilder
    .Entity<Blog>()
    .Property(e => e.Id)
    .HasField("_id");
```

## <a name="field-only-property-names-should-match-the-field-name"></a>Los nombres de propiedades de solo campo deben coincidir con el nombre del campo

Este cambio se introdujo en EF Core 3.0 (versión preliminar 4).

**Comportamiento anterior**

Antes de EF Core 3.0, una propiedad podía especificarse con un valor de cadena y, si no había ninguna propiedad con ese nombre en el tipo CLR, EF Core intentaba hacerla coincidir con un campo mediante reglas de convención.
```C#
private class Blog
{
    private int _id;
    public string Name { get; set; }
}
```
```C#
modelBuilder
    .Entity<Blog>()
    .Property("Id");
```

**Comportamiento nuevo**

A partir de EF Core 3.0, una propiedad de solo campo debe coincidir exactamente con el nombre del campo.

```C#
modelBuilder
    .Entity<Blog>()
    .Property("_id");
```

**Por qué**

Este cambio se realizó para evitar el uso del mismo campo para dos propiedades con nombres similares. También hace que las reglas de coincidencia para propiedades solo de campo sean las mismas que para las propiedades asignadas a propiedades CLR.

**Mitigaciones**

Las propiedades solo de campo deberían tener el mismo nombre que el campo al que están asignadas.
En una próxima versión preliminar de EF Core 3.0 tenemos planeado volver a habilitar la configuración explícita de un nombre de campo distinto al nombre de la propiedad:

```C#
modelBuilder
    .Entity<Blog>()
    .Property("Id")
    .HasField("_id");
```

## <a name="adddbcontextadddbcontextpool-no-longer-call-addlogging-and-addmemorycache"></a>AddDbContext/AddDbContextPool ya no llaman a AddLogging ni a AddMemoryCache

[Problema de seguimiento n.° 14756](https://github.com/aspnet/EntityFrameworkCore/issues/14756)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 4).

**Comportamiento anterior**

Antes de EF Core 3.0, llamar a `AddDbContext` o a `AddDbContextPool` también podría registrar los servicios de almacenamiento en caché y de registro con D.I. a través de llamadas a [AddLogging](https://docs.microsoft.com/dotnet/api/microsoft.extensions.dependencyinjection.loggingservicecollectionextensions.addlogging) y a [AddMemoryCache](https://docs.microsoft.com/dotnet/api/microsoft.extensions.dependencyinjection.memorycacheservicecollectionextensions.addmemorycache).

**Comportamiento nuevo**

A partir de EF Core 3.0, `AddDbContext` y `AddDbContextPool` ya no registrarán estos servicios con inserción de dependencias (DI).

**Por qué**

EF Core 3.0 no requiere que estos servicios estén en el contenedor de inserción de dependencias de la aplicación. Pero si `ILoggerFactory` se registra en el contenedor de DI de la aplicación, EF Core lo empezará a usar de todos modos.

**Mitigaciones**

Si la aplicación necesita estos servicios, regístrelos de manera explícita con el contenedor de DI mediante [AddLogging](https://docs.microsoft.com/dotnet/api/microsoft.extensions.dependencyinjection.loggingservicecollectionextensions.addlogging) o [AddMemoryCache](https://docs.microsoft.com/dotnet/api/microsoft.extensions.dependencyinjection.memorycacheservicecollectionextensions.addmemorycache).

## <a name="dbcontextentry-now-performs-a-local-detectchanges"></a>Ahora DbContext.Entry realiza una operación DetectChanges local

[Problema de seguimiento n.º 13552](https://github.com/aspnet/EntityFrameworkCore/issues/13552)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 3).

**Comportamiento anterior**

Antes de EF Core 3.0, la llamada a `DbContext.Entry` provocaba que se detectaran cambios para todas las entidades con seguimiento.
Esto garantizaba que el estado expuesto en `EntityEntry` estuviera actualizado.

**Comportamiento nuevo**

A partir de EF Core 3.0, ahora la llamada a `DbContext.Entry` solo intenta detectar cambios en la entidad dada y cualquier entidad de seguridad relacionada con ella de la que se haya realizado el seguimiento.
Esto significa que es posible que la llamada a este método no haya detectado otros cambios, lo que podría tener implicaciones en el estado de la aplicación.

Observe que si `ChangeTracker.AutoDetectChangesEnabled` se establece en `false` incluso esta detección de cambios local se deshabilitará.

Otros métodos que provocan la detección de cambios (como `ChangeTracker.Entries` y `SaveChanges`) siguen provocando una acción `DetectChanges` completa de todas las entidades de las que se realiza el seguimiento.

**Por qué**

Este cambio se ha realizado para mejorar el rendimiento predeterminado del uso de `context.Entry`.

**Mitigaciones**

Llame a `ChgangeTracker.DetectChanges()` de forma explícita antes de llamar a `Entry` para garantizar el comportamiento anterior a la versión 3.0.

## <a name="string-and-byte-array-keys-are-not-client-generated-by-default"></a>El cliente no genera las claves de matriz de cadena y byte de forma predeterminada

[Problema de seguimiento n.º 14617](https://github.com/aspnet/EntityFrameworkCore/issues/14617)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 4).

**Comportamiento anterior**

Antes de EF Core 3.0, se podían usar las propiedades de clave `string` y `byte[]` sin tener que establecer de forma explícita un valor distinto de NULL.
En ese caso, el valor de clave se generaba en el cliente como un GUID, que se serializaba en bytes para `byte[]`.

**Comportamiento nuevo**

A partir de EF Core 3.0, se iniciará una excepción en la que indica que no se ha establecido ningún valor de clave.

**Por qué**

Este cambio se ha realizado porque los valores `string`/`byte[]` generados por el cliente no suelen ser útiles, y el comportamiento predeterminado dificultaba razonar sobre los valores de clave generados de una forma habitual.

**Mitigaciones**

Se puede obtener el comportamiento anterior a la versión 3.0 si se especifica de forma explícita que las propiedades de clave deben usar los valores generados si no se establece ningún otro valor distinto de NULL.
Por ejemplo, con la API fluida:

```C#
modelBuilder
    .Entity<Blog>()
    .Property(e => e.Id)
    .ValueGeneratedOnAdd();
```

O bien con anotaciones de datos:

```C#
[DatabaseGenerated(DatabaseGeneratedOption.Identity)]
public string Id { get; set; }
```

## <a name="iloggerfactory-is-now-a-scoped-service"></a>Ahora ILoggerFactory es un servicio con ámbito

[Problema de seguimiento n.º 14698](https://github.com/aspnet/EntityFrameworkCore/issues/14698)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 3).

**Comportamiento anterior**

Antes de EF Core 3.0, `ILoggerFactory` se registraba como un servicio de singleton.

**Comportamiento nuevo**

A partir de EF Core 3.0, `ILoggerFactory` ahora se registra como con ámbito.

**Por qué**

Este cambio se ha realizado para permitir la asociación de un registrador con una instancia de `DbContext`, lo que habilita otras funciones y quita algunos casos de comportamiento patológico como un aumento vertiginoso de los proveedores de servicios internos.

**Mitigaciones**

Este cambio no debería afectar al código de la aplicación a menos que registre y use servicios personalizados en el proveedor de servicios internos de EF Core.
Esto no es habitual.
En estos casos, la mayoría de los elementos seguirá funcionando, pero cualquier servicio de singleton que dependiera de `ILoggerFactory` tendrá que cambiarse para obtener la interfaz `ILoggerFactory` de otra forma.

Si experimenta situaciones como esta, registre un problema en el [rastreador de problemas de GitHub de EF Core](https://github.com/aspnet/EntityFrameworkCore/issues) para hacernos saber cómo usa `ILoggerFactory`, para que podamos comprender mejor cómo evitar esta interrupción en el futuro.

## <a name="idbcontextoptionsextensionwithdebuginfo-merged-into-idbcontextoptionsextension"></a>IDbContextOptionsExtensionWithDebugInfo se ha fusionado en IDbContextOptionsExtension

[Problema de seguimiento n.º 13552](https://github.com/aspnet/EntityFrameworkCore/issues/13552)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 3).

**Comportamiento anterior**

`IDbContextOptionsExtensionWithDebugInfo` era una interfaz opcional adicional ampliada de `IDbContextOptionsExtension` para evitar realizar cambios importantes en la interfaz durante el ciclo de versiones 2.x.

**Comportamiento nuevo**

Ahora las interfaces se combinan en `IDbContextOptionsExtension`.

**Por qué**

Este cambio se ha realizado porque las interfaces son conceptualmente una.

**Mitigaciones**

Cualquier implementación de `IDbContextOptionsExtension` tendrá que actualizarse para admitir el miembro nuevo.

## <a name="lazy-loading-proxies-no-longer-assume-navigation-properties-are-fully-loaded"></a>En los proxies de carga diferida ya no se supone que las propiedades de navegación están totalmente cargadas

[Problema de seguimiento n.º 12780](https://github.com/aspnet/EntityFrameworkCore/issues/12780)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 4).

**Comportamiento anterior**

Antes de EF Core 3.0, una vez que se eliminaba `DbContext` no había ninguna forma de saber si una determinada propiedad de navegación de una entidad obtenida de ese contexto se había cargado completamente o no.
En su lugar, los proxies asumían que una navegación de referencia se cargaba si tenía un valor distinto de NULL, y que una navegación de colección se cargaba si no estaba vacía.
En estos casos, el intento de carga diferida era no operativo.

**Comportamiento nuevo**

A partir de EF Core 3.0, los proxies realizan el seguimiento de si una propiedad de navegación se carga o no.
Esto significa que el intento de acceder a una propiedad de navegación que se carga después de que se haya eliminado el contexto siempre será no operativo, incluso cuando la navegación cargada está vacía o es NULL.
Por el contrario, el intento de acceder a una propiedad de navegación que no está cargada iniciará una excepción si el contexto se ha eliminado, incluso si la propiedad de navegación es una colección no vacía.
Si se produce esta situación, significa que el código de aplicación está intentando usar la carga diferida en un momento no válido y que se debe cambiar la aplicación para que lo no haga.

**Por qué**

Este cambio se ha realizado para que el comportamiento sea coherente y correcto cuando se intenta la carga diferida de una instancia de `DbContext` eliminada.

**Mitigaciones**

Actualice el código de la aplicación para que no intente la carga diferida con un contexto eliminado, o bien configúrelo para que sea no operativo, como se describe en el mensaje de la excepción.

## <a name="excessive-creation-of-internal-service-providers-is-now-an-error-by-default"></a>La creación excesiva de proveedores de servicios internos ahora es un error de forma predeterminada

[Problema de seguimiento n.º 10236](https://github.com/aspnet/EntityFrameworkCore/issues/10236)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 3).

**Comportamiento anterior**

Antes de EF Core 3.0, se registraba una advertencia para una aplicación que creaba un número patológico de proveedores de servicios internos.

**Comportamiento nuevo**

A partir de EF Core 3.0, ahora esta advertencia se considera un error y se inicia una excepción. 

**Por qué**

Este cambio se ha realizado para controlar mejor el código de la aplicación mediante la exposición de este caso patológico de una forma más explícita.

**Mitigaciones**

Cuando se produce este error, la acción más adecuada consiste en comprender la causa raíz y detener la creación de tantos proveedores de servicios internos.
Pero el error se puede convertir en una advertencia (u omitirse) mediante configuración en `DbContextOptionsBuilder`.
Por ejemplo:

```C#
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .ConfigureWarnings(w => w.Log(CoreEventId.ManyServiceProvidersCreatedWarning));
}
```

## <a name="new-behavior-for-hasonehasmany-called-with-a-single-string"></a>Comportamiento nuevo de HasOne/HasMany llamado con una sola cadena

[Problema de seguimiento n.° 9171](https://github.com/aspnet/EntityFrameworkCore/issues/9171)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 4).

**Comportamiento anterior**

Antes de EF Core 3.0, el código para llamar a `HasOne` o `HasMany` con una cadena se interpretaba de manera confusa.
Por ejemplo:
```C#
modelBuilder.Entity<Samurai>().HasOne("Entrance").WithOne();
```

El código parece relacionar `Samurai` con otro tipo de entidad mediante la propiedad de navegación `Entrance`, que puede ser privada.

En realidad, este código intenta crear una relación con algún tipo de entidad denominada `Entrance` sin ninguna propiedad de navegación.

**Comportamiento nuevo**

A partir de EF Core 3.0, el código anterior ahora hace lo que parecía que debía hacer antes.

**Por qué**

El comportamiento anterior era muy confuso, especialmente al leer el código de configuración y al buscar errores.

**Mitigaciones**

Esto solo interrumpirá las aplicaciones que configuran de manera explícita las relaciones con cadenas para nombres de tipos y sin especificar explícitamente la propiedad de navegación.
Esto no es habitual.
El comportamiento anterior se puede obtener al pasar de manera explícita `null` para el nombre de la propiedad de navegación.
Por ejemplo:

```C#
modelBuilder.Entity<Samurai>().HasOne("Some.Entity.Type.Name", null).WithOne();
```

## <a name="the-return-type-for-several-async-methods-has-been-changed-from-task-to-valuetask"></a>El tipo de valor devuelto para varios métodos asincrónicos se ha cambiado de Task a ValueTask

[Problema de seguimiento n.º 15184](https://github.com/aspnet/EntityFrameworkCore/issues/15184)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 4).

**Comportamiento anterior**

Antes, los siguientes métodos asincrónicos devolvían `Task<T>`:

* `DbContext.FindAsync()`
* `DbSet.FindAsync()`
* `DbContext.AddAsync()`
* `DbSet.AddAsync()`
* `ValueGenerator.NextValueAsync()` (y las clases derivadas)

**Comportamiento nuevo**

Dichos métodos ahora devuelven `ValueTask<T>` durante el mismo `T` que antes.

**Por qué**

Este cambio reduce el número de asignaciones de montones que se producen al invocar estos métodos, lo que mejora el rendimiento general.

**Mitigaciones**

Las aplicaciones que simplemente esperen las API anteriores solo necesitan recompilarse, sin que sea necesario realizar cambios en el código fuente.
Un uso más complejo (p. ej., pasar el valor `Task` devuelto a `Task.WhenAny()`) normalmente requiere que el valor `ValueTask<T>` devuelto se convierta en `Task<T>` mediante una llamada a `AsTask()` en él.
Tenga en cuenta que esto niega la reducción de asignación que implica este cambio.

## <a name="the-relationaltypemapping-annotation-is-now-just-typemapping"></a>La anotación Relational:TypeMapping ahora es simplemente TypeMapping

[Problema de seguimiento n.º 9913](https://github.com/aspnet/EntityFrameworkCore/issues/9913)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 2).

**Comportamiento anterior**

El nombre de anotación para las anotaciones de asignación de tipos era "Relational:TypeMapping".

**Comportamiento nuevo**

Ahora, el nombre de anotación para las anotaciones de asignación de tipos es "TypeMapping".

**Por qué**

Ahora, las asignaciones de tipos se usan para algo más que solo para proveedores de bases de datos relacionales.

**Mitigaciones**

Esto solo interrumpirá a las aplicaciones que acceden directamente a la asignación de tipos como una anotación, lo que no es habitual.
La acción más apropiada para corregir es usar la superficie de API para acceder a las asignaciones de tipos en lugar de usar directamente la anotación.

## <a name="totable-on-a-derived-type-throws-an-exception"></a>ToTable en un tipo derivado inicia una excepción 

[Problema de seguimiento n.º 11811](https://github.com/aspnet/EntityFrameworkCore/issues/11811)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 3).

**Comportamiento anterior**

Antes de EF Core 3.0, la llamada a `ToTable()` en un tipo derivado se omitía, ya que la única estrategia asignación de herencia era TPH, lo que no es válido. 

**Comportamiento nuevo**

A partir de EF Core 3.0, y en preparación para agregar compatibilidad con TPT y TPC en una versión posterior, ahora la llamada a `ToTable()` en un tipo derivado iniciará una excepción para evitar un cambio de asignación inesperado en el futuro.

**Por qué**

En la actualidad no se considera válido asignar un tipo derivado a otra tabla.
Este cambio evita interrupciones en el futuro, cuando se convierta en una operación válida.

**Mitigaciones**

Quite todos los intentos de asignar tipos derivados a otras tablas.

## <a name="forsqlserverhasindex-replaced-with-hasindex"></a>ForSqlServerHasIndex se ha reemplazado por HasIndex 

[Problema de seguimiento n.º 12366](https://github.com/aspnet/EntityFrameworkCore/issues/12366)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 3).

**Comportamiento anterior**

Antes de EF Core 3.0, `ForSqlServerHasIndex().ForSqlServerInclude()` proporcionaba una manera de configurar las columnas que se usaban con `INCLUDE`.

**Comportamiento nuevo**

A partir de EF Core 3.0, ya se admite el uso de `Include` en un índice en el nivel relacional.
Use `HasIndex().ForSqlServerInclude()`.

**Por qué**

Este cambio se ha realizado para consolidar la API para índices con `Include` en un mismo lugar para todos los proveedores de base de datos.

**Mitigaciones**

Use la API nueva, como se ha mostrado anteriormente.

## <a name="metadata-api-changes"></a>Cambios en la API de metadatos

[Problema de seguimiento n.º 214](https://github.com/aspnet/EntityFrameworkCore/issues/214)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 4).

**Comportamiento nuevo**

Las siguientes propiedades se han convertido en métodos de extensión:

* `IEntityType.QueryFilter` -> `GetQueryFilter()`
* `IEntityType.DefiningQuery` -> `GetDefiningQuery()`
* `IProperty.IsShadowProperty` -> `IsShadowProperty()`
* `IProperty.BeforeSaveBehavior` -> `GetBeforeSaveBehavior()`
* `IProperty.AfterSaveBehavior` -> `GetAfterSaveBehavior()`

**Por qué**

Este cambio simplifica la implementación de las interfaces mencionadas anteriormente.

**Mitigaciones**

Use los nuevos métodos de extensión.

## <a name="provider-specific-metadata-api-changes"></a>Cambios en la API de metadatos específicos del proveedor

[Problema de seguimiento n.º 214](https://github.com/aspnet/EntityFrameworkCore/issues/214)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 6).

**Comportamiento nuevo**

Los métodos de extensión específicos del proveedor se simplificarán:

* `IProperty.Relational().ColumnName` -> `IProperty.GetColumnName()`
* `IEntityType.SqlServer().IsMemoryOptimized` -> `IEntityType.GetSqlServerIsMemoryOptimized()`
* `PropertyBuilder.UseSqlServerIdentityColumn()` -> `PropertyBuilder.ForSqlServerUseIdentityColumn()`

**Por qué**

Este cambio simplifica la implementación de los métodos de extensión mencionados anteriormente.

**Mitigaciones**

Use los nuevos métodos de extensión.

## <a name="ef-core-no-longer-sends-pragma-for-sqlite-fk-enforcement"></a>EF Core ya no envía pragma para el cumplimiento de SQLite FK

[Problema de seguimiento n.º 12151](https://github.com/aspnet/EntityFrameworkCore/issues/12151)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 3).

**Comportamiento anterior**

Antes de EF Core 3.0, EF Core enviaba `PRAGMA foreign_keys = 1` cuando se abría una conexión con SQLite.

**Comportamiento nuevo**

A partir de EF Core 3.0, EF Core ya no envía `PRAGMA foreign_keys = 1` cuando se abre una conexión con SQLite.

**Por qué**

Este cambio se ha realizado porque en EF Core se usa `SQLitePCLRaw.bundle_e_sqlite3` de forma predeterminada, lo que a su vez significa que el cumplimiento de CD está activado de forma predeterminada y no es necesario habilitarlo explícitamente cada vez que se abra una conexión.

**Mitigaciones**

Las claves externas se habilitan de forma predeterminada en SQLitePCLRaw.bundle_e_sqlite3, que en EF Core se usa de forma predeterminada.
Para otros casos, las claves externas se pueden habilitar mediante la especificación de `Foreign Keys=True` en la cadena de conexión.

## <a name="microsoftentityframeworkcoresqlite-now-depends-on-sqlitepclrawbundleesqlite3"></a>Microsoft.EntityFrameworkCore.Sqlite ahora depende de SQLitePCLRaw.bundle_e_sqlite3

**Comportamiento anterior**

Antes de EF Core 3.0, en EF Core se usaba `SQLitePCLRaw.bundle_green`.

**Comportamiento nuevo**

A partir de EF Core 3.0, en EF Core se usa `SQLitePCLRaw.bundle_e_sqlite3`.

**Por qué**

Este cambio se ha realizado para que la versión de SQLite que se usa en iOS sea coherente con otras plataformas.

**Mitigaciones**

Para usar la versión nativa de SQLite en iOS, configure `Microsoft.Data.Sqlite` para usar otra agrupación `SQLitePCLRaw`.

## <a name="guid-values-are-now-stored-as-text-on-sqlite"></a>Almacenamiento de valores GUID como TEXT en SQLite

[Problema de seguimiento n.º 15078](https://github.com/aspnet/EntityFrameworkCore/issues/15078)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 4).

**Comportamiento anterior**

Antes los valores GUID se almacenaban como valores BLOB en SQLite.

**Comportamiento nuevo**

Ahora, los valores GUID se almacenan como TEXT.

**Por qué**

El formato binario de los GUID no está normalizado. El almacenamiento de los valores como TEXT mejora la compatibilidad de la base de datos con otras tecnologías.

**Mitigaciones**

Puede migrar las bases de datos existentes al nuevo formato ejecutando SQL de la siguiente forma.

``` sql
UPDATE MyTable
SET GuidColumn = hex(substr(GuidColumn, 4, 1)) ||
                 hex(substr(GuidColumn, 3, 1)) ||
                 hex(substr(GuidColumn, 2, 1)) ||
                 hex(substr(GuidColumn, 1, 1)) || '-' ||
                 hex(substr(GuidColumn, 6, 1)) ||
                 hex(substr(GuidColumn, 5, 1)) || '-' ||
                 hex(substr(GuidColumn, 8, 1)) ||
                 hex(substr(GuidColumn, 7, 1)) || '-' ||
                 hex(substr(GuidColumn, 9, 2)) || '-' ||
                 hex(substr(GuidColumn, 11, 6))
WHERE typeof(GuidColumn) == 'blob';
```

En EF Core, también puede seguir usando el comportamiento anterior configurando un convertidor de valores en estas propiedades.

``` csharp
modelBuilder
    .Entity<MyEntity>()
    .Property(e => e.GuidProperty)
    .HasConversion(
        g => g.ToByteArray(),
        b => new Guid(b));
```

Microsoft.Data.Sqlite sigue siendo capaz de leer valores GUID de ambas columnas BLOB y TEXT. Sin embargo, dado que el formato predeterminado de los parámetros y las constantes ha cambiado, seguramente deberá realizar alguna acción en la mayoría de casos que impliquen el uso de valores GUID.

## <a name="char-values-are-now-stored-as-text-on-sqlite"></a>Ahora los valores char se almacenan como TEXT en SQLite

[Problema de seguimiento n.º 15020](https://github.com/aspnet/EntityFrameworkCore/issues/15020)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 4).

**Comportamiento anterior**

Anteriormente los valores char se almacenaban como valores INTEGER en SQLite. Por ejemplo, un valor char de *A* se almacenaba como el valor entero 65.

**Comportamiento nuevo**

Ahora, los valores char se almacenan como TEXT.

**Por qué**

El almacenamiento de valores como TEXT es más natural y mejora la compatibilidad de la base de datos con otras tecnologías.

**Mitigaciones**

Puede migrar las bases de datos existentes al nuevo formato ejecutando SQL de la siguiente forma.

``` sql
UPDATE MyTable
SET CharColumn = char(CharColumn)
WHERE typeof(CharColumn) = 'integer';
```

En EF Core, también puede seguir usando el comportamiento anterior configurando un convertidor de valores en estas propiedades.

``` csharp
modelBuilder
    .Entity<MyEntity>()
    .Property(e => e.CharProperty)
    .HasConversion(
        c => (long)c,
        i => (char)i);
```

Microsoft.Data.Sqlite también puede leer valores de caracteres tanto de columnas INTEGER como de columnas TEXT, por lo que es posible que no deba hacer nada dependiendo de su caso.

## <a name="migration-ids-are-now-generated-using-the-invariant-cultures-calendar"></a>Ahora los id. de migración se generan usando el calendario de la referencia cultural invariable

[Problema de seguimiento n.º 12978](https://github.com/aspnet/EntityFrameworkCore/issues/12978)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 4).

**Comportamiento anterior**

Los identificadores de migración se generaban de forma involuntaria con el calendario de la referencia cultural actual.

**Comportamiento nuevo**

Ahora los id. de migración siempre se generan usando el calendario de la referencia cultural invariable (gregoriano).

**Por qué**

El orden de las migraciones es importante al actualizar la base de datos o al solucionar conflictos de combinación. Al usar el calendario invariable, se evitan problemas de ordenación que pueden producirse si los miembros del equipo tienen distintos calendarios del sistema.

**Mitigaciones**

Esta cambio afecta a todas las personas que usan un calendario no gregoriano en el que el año sea superior al del calendario gregoriano (como el calendario budista tailandés). Los id. de migración existentes deberán actualizarse para que las migraciones nuevas se ordenen después de las existentes.

Puede ver el id. de migración en el atributo Migration de los archivos de diseñador de la migración.

``` diff
 [DbContext(typeof(MyDbContext))]
-[Migration("25620318122820_MyMigration")]
+[Migration("20190318122820_MyMigration")]
 partial class MyMigration
 {
```

También debe actualizarse la tabla de historial de migraciones.

``` sql
UPDATE __EFMigrationsHistory
SET MigrationId = CONCAT(LEFT(MigrationId, 4)  - 543, SUBSTRING(MigrationId, 4, 150))
```

## <a name="logquerypossibleexceptionwithaggregateoperator-has-been-renamed"></a>Cambio de nombre de LogQueryPossibleExceptionWithAggregateOperator

[Problema de seguimiento n.º 10985](https://github.com/aspnet/EntityFrameworkCore/issues/10985)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 4).

**Cambio**

Se ha cambiado el nombre de `RelationalEventId.LogQueryPossibleExceptionWithAggregateOperator` a `RelationalEventId.LogQueryPossibleExceptionWithAggregateOperatorWarning`.

**Por qué**

Conviene alinear el nombre de este evento de advertencia con el del resto de eventos de advertencia.

**Mitigaciones**

Use el nuevo nombre. (Tenga en cuenta que el número de id. evento sigue siendo el mismo).

## <a name="clarify-api-for-foreign-key-constraint-names"></a>Clarificación de la API para nombres de restricciones de claves externas

[Problema de seguimiento n.º 10730](https://github.com/aspnet/EntityFrameworkCore/issues/10730)

Este cambio se introdujo en EF Core 3.0 (versión preliminar 4).

**Comportamiento anterior**

Antes de EF Core 3.0, se utilizaba simplemente el término "nombre" para hacer referencia a los nombres de las restricciones de claves externas. Por ejemplo:

```C#
var constraintName = myForeignKey.Name;
```

**Comportamiento nuevo**

A partir de EF Core 3.0, el término con el que se hace referencia a los nombres de las restricciones de claves externas es "nombre de la restricción". Por ejemplo:

```C#
var constraintName = myForeignKey.ConstraintName;
```

**Por qué**

Este cambio permite mejorar la coherencia relativa a la nomenclatura en este aspecto y aclarar que se trata del nombre de una restricción de clave externa, y no del de la columna o propiedad en la que está definida la clave externa.

**Mitigaciones**

Use el nuevo nombre.
