---
name: Composer Migration
dcc_uri: dev/rules/composer_migration
description: Rules for converting and migrating WSBCC XML operation definitions (CCDSEServerOperation) to framework-free native Java. No IBM, WebSphere, or WSBCC dependencies in generated code.
version: '1.0.0'
dcc_tags:
  - dev
  - backend
  - java
  - composer
  - wsbcc
---

# Composer Convert Rules

These rules govern the conversion of WSBCC XML operation definitions into plain, framework-free Java. The output must have zero dependency on IBM WebSphere, WSBCC, or any reflection-driven tag execution infrastructure. Every concept from the XML definition has a direct Java equivalent — the rules below define those mappings precisely.

---

## Guiding Principle

The WSBCC XML definition is a declarative description of a service. The conversion produces an equivalent imperative Java implementation. Every structural element of the XML maps to a specific Java construct. Do not produce generic utilities that mimic the WSBCC runtime — produce readable, maintainable business code.

---

## CCDSEServerOperation → Service Class

Each `CCDSEServerOperation` becomes a single Java service class.

- Class name: the value of the `id` attribute, suffixed with `Service` (e.g. `id="GetLinksOp"` → `GetLinksOpService`).
- The class has a single public method: `execute(OperationContext context)` where `OperationContext` is the converted context class (see below).
- The class is annotated with nothing framework-specific — it is a plain Java class with constructor-injected dependencies.
- All `opStep` implementing classes referenced via `implClass` become constructor-injected dependencies of this service class. Never instantiate them inline.
- The class must not use reflection, XML parsing, or any WSBCC infrastructure at runtime.

```java
// CCDSEServerOperation id="GetLinksOp" → 
public class GetLinksOpService {
    private final GetClientLinksSP getClientLinks;
    private final MapErrorByValueAlways mapError;

    public GetLinksOpService(GetClientLinksSP getClientLinks,
                             MapErrorByValueAlways mapError) {
        this.getClientLinks = getClientLinks;
        this.mapError = mapError;
    }

    public GetLinksRsDto execute(GetLinksContext context) { ... }
}
```

---

## opStep Chain → Step Routing Methods

The `opStep` sequence with `on{X}Do` / `onOtherDo` routing is a state machine. Convert it to a private routing method within the service class using explicit `switch` expressions.

- Each `opStep` with an `implClass` becomes a call to the injected dependency's `execute()` method.
- The `on{X}Do` attributes become `case` branches in a `switch` expression on the returned `int`.
- `onOtherDo` becomes the `default` branch.
- `onOtherDo="end"` and `on0Do="end"` mean the step terminates the chain — return the result from the method.
- `opStep` entries with no `implClass` that only route to `end` (e.g. `OperationFailed`) become a thrown exception or an error result record — never a no-op.
- Custom attributes on an `opStep` (e.g. `ErrorCategory="10"`) that were previously injected via reflection must be passed as constructor arguments or factory method parameters to the step class — not set via setters.

```java
// opStep chain →
private GetLinksRsDto runSteps(GetLinksContext context) {
    return switch (getClientLinks.execute(context)) {
        case 0  -> buildResponse(context);
        case 4  -> mapError.execute(context, 10, 0);  // ErrorDataNotFound
        case 5  -> mapError.execute(context, 20, 0);  // ErrorInvalidLinks
        default -> throw new OperationFailedException("GetLinksOp failed");
    };
}
```

---

## fmtDef → Request and Response Records

Each `fmtDef` becomes a Java `record` (Java 16+). The `id` attribute is the record name.

- Request formats (referenced as `csRequestFormat`) → input record, suffixed `RqDto`.
- Response formats (referenced as `csReplyFormat`) → output record, suffixed `RsDto`.
- Each child format tag maps to a field in the record according to the type mapping table below.
- `CCXML` wrapper tags with `transparent="true"` do not produce a wrapper record — their children are flattened into the parent record.
- `CCXML` tags with `transparent="false"` or no `transparent` attribute produce a nested record.
- `CCTcoll` with `times="*"` maps to `List<T>` where `T` is the record type of the child `CCXML`.

```java
// fmtDef id="GetLinksRQFmt" →
public record GetLinksRqDto(String clientId, String clientName) {}

// fmtDef id="GetLinksRSFmt" →
public record GetLinksRsDto(
    String clientId,
    String clientName,
    List<DocDataDto> linksBlock
) {}

public record DocDataDto(
    String docId,
    String flag,
    String linkDescription,   // CCTableFormat → String (resolved value)
    String storeDate,
    BigDecimal maxAccess,     // NumberFormat → BigDecimal
    LocalDate writtenAt       // CCDate → LocalDate
) {}
```

---

## Format Tag Type Mapping

Convert each format tag to the most appropriate native Java type. Do not use `String` for everything — model the type the field actually represents.

| WSBCC Tag | Java Type | Notes |
|---|---|---|
| `CCString` | `String` | Direct mapping |
| `CCDate` | `LocalDate` / `LocalDateTime` | Use `pattern` attribute to select the correct `DateTimeFormatter`; `times="*"` means the format handles a collection |
| `CCBoolean` | `boolean` / `Boolean` | Map `"yes"`/`"no"` strings to `true`/`false` |
| `CCInteger` | `int` / `Integer` | Direct mapping |
| `NumberFormat` | `BigDecimal` | Ignore `showThousandsSep` / `showDecimalsSep` — those are presentation concerns handled at the serialisation layer |
| `CCTableFormat` | `String` | The resolved lookup value; document `fromTable`, `fromColumn`, `keyValue` as a comment on the field for traceability |
| `CCTcoll` with `times="*"` | `List<T>` | `T` is the record type of the child element |
| `CCXML` transparent | _(flatten)_ | No wrapper type; promote children to parent record |
| `CCXML` non-transparent | Nested record | Create a named nested record |
| `nilDecorator` | _(omit)_ | Null-decoration is a WSBCC rendering concern; make the preceding field `Optional<T>` if nullability is meaningful |

---

## context → Context Class

Each `context` tag becomes a plain Java class (not a record — it is mutable shared state passed between steps).

- Class name: the `id` attribute value (e.g. `id="GetLinksCtxt"` → `GetLinksContext`).
- The context holds the parsed request DTO and accumulates results as steps execute.
- Fields are package-private or have explicit getters/setters — no public mutable fields.
- If `parent` is not `"nil"`, the converted context class extends the parent context class.
- The context class must not hold any WSBCC or WebSphere types.

```java
// context id="GetLinksCtxt" parent="nil" →
public class GetLinksContext {
    private GetLinksRqDto request;
    private GetLinksRsDto response;

    // getters and setters
}
```

---

## refFormat → DTO Reuse

`refFormat` is a reference to an existing `fmtDef`. In the converted Java code:

- Do not duplicate the record definition. Reference the same record class.
- The `name` attribute (e.g. `csRequestFormat`, `csReplyFormat`) becomes a local variable name or method parameter name in the service class where it is used.

---

## opStep implClass → Step Interface and Implementation

Each unique `implClass` value referenced in an `opStep` must become a Java class that implements a common step interface.

- Define a shared interface: `OperationStep<C>` with a single method `int execute(C context)`.
- Each `implClass` (e.g. `db.GetClientLinksSP`, `operations.MapErrorByValueAlways`) becomes a concrete class implementing this interface.
- Custom attributes that were previously set via reflection (e.g. `ErrorCategory`, `ErrorNumber`) become constructor parameters of the concrete class — never setter methods.
- The step class must only depend on the context and its constructor-injected collaborators (data sources, external clients, etc.). No static access, no service locators.

```java
public interface OperationStep<C> {
    int execute(C context);
}

// implClass="operations.MapErrorByValueAlways" ErrorCategory="10" ErrorNumber="0" →
public class MapErrorByValueAlways implements OperationStep<GetLinksContext> {
    private final int errorCategory;
    private final int errorNumber;

    public MapErrorByValueAlways(int errorCategory, int errorNumber) {
        this.errorCategory = errorCategory;
        this.errorNumber = errorNumber;
    }

    @Override
    public int execute(GetLinksContext context) { ... }
}
```

---

## XML Serialisation / Deserialisation

The WSBCC framework handled XML parsing and formatting transparently. In the converted code this must be explicit, but it must be kept at the boundary — not inside business logic.

- Parse the inbound XML request into the request DTO at the entry point of the service (e.g. in a controller or adapter layer). Do not pass raw XML strings into the service class or step classes.
- Serialise the response DTO to XML at the exit point. Do not build XML strings inside step classes.
- Use a standard Java XML library (JAXB, Jackson with XML module, or similar) for serialisation. Document the choice with a brief comment. Do not write manual XML string concatenation.
- `CCDate` parsing: translate the `pattern` attribute directly to a `DateTimeFormatter` constant. The formatter is a concern of the serialisation layer, not the domain record.

---

## What Must Not Appear in Generated Code

- No imports from `com.ibm.*`, `com.websphere.*`, or any WSBCC package.
- No use of `Class.forName()`, `Method.invoke()`, or any reflection-based attribute setting.
- No XML parsing inside step classes or the service class — only in the entry/exit serialisation layer.
- No `dse.ini` lookups or runtime tag-to-class resolution.
- No setter methods named to match XML attribute names (the reflection contract is gone — use constructors).
- No `String` fields for values that have a clear Java type (`boolean`, `LocalDate`, `BigDecimal`, etc.).
