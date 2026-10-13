# ActionScript Compiler 2.0 Backward Compatibility

This document describes the known backward compatibility issues between
ActionScript Compiler 2.0 (ASC 2.0) and the legacy compiler. The
document also includes features that are no longer available in ASC 2.0
and deprecated features that are no longer supported.

### Severity of Change Key

- **Stricter:** ASC 2.0 emits an error or warning where legacy compilers
  did not
- **More lenient:** ASC 2.0 accepts code that legacy compilers did not
- **Change in code generation with diagnostic:** ASC 2.0 emits code that
  behaves differently than code emitted by the legacy compiler and
  issues an error or warning
- **Silent change in code generation:** ASC 2.0 emits code that behaves
  differently than code emitted by the legacy compiler without issuing
  an error or warning
- **Diagnostic change:** ASC 2.0 emits different diagnostics than the
  legacy compiler

## Compatibility Behavior Changes

These changes are for greater conformance with the ActionScript 3.0
Language Specification. The legacy compiler had bugs that allowed code
to circumvent the language rules, these bugs have been addressed in ASC
2.0.

This document includes information related to:

[ActionScript 3.0 Language](#actionscript-30-language)

[Flash Builder Changes](#flash-builder-changes)

### ActionScript 3.0 Language

<table>
<colgroup>
<col style="width: 20%" />
<col style="width: 20%" />
<col style="width: 20%" />
<col style="width: 20%" />
<col style="width: 20%" />
</colgroup>
<tbody>
<tr>
<th>Behavior change</th>
<th>Behavior in legacy compiler</th>
<th>Behavior in ASC 2.0</th>
<th>When will you encounter this issue?</th>
<th>Severity of change</th>
</tr>
<tr>
<td><p>'<code>public</code>', '<code>private</code>', and
'<code>protected</code>' are now keywords</p></td>
<td><p>'<code>public</code>', '<code>private</code>', and
'<code>protected</code>' were contextually reserved identifiers. They
could be used in some places, such as in a member expression.</p></td>
<td><p>'<code>public</code>', '<code>private</code>', and
'<code>protected</code>' are now keywords, and will be interpreted as
such no matter where they are used. That is, certain usages allowed by
the legacy compiler, like in a member expression, will now be
disallowed.</p></td>
<td><p>When you use '<code>public</code>' in a member
expression:</p>
<p> </p>
<p><code>a.public = "foo";</code></p></td>
<td><p>Stricter</p></td>
</tr>
<tr>
<td><p>'<code>internal</code>' is now a keyword, and is
treated the same as '<code>public</code>', '<code>protected</code>', or
'<code>private</code>'</p></td>
<td><p>'<code>internal</code>' was not a keyword and could
be used as an identifier, such as the name of a variable.</p></td>
<td><p>'<code>internal</code>' is now a keyword and a parse
error is generated if you try to use it as an identifier.</p></td>
<td><p>When you use 'internal' as an identifier, such as in
a var declaration:</p>
<p><code>var internal:String</code>;</p></td>
<td><p>Stricter</p></td>
</tr>
<tr>
<td><p>Curly braces are required for config blocks</p></td>
<td><p>You can define code in config blocks as follows:</p>
<p><code>CONFIG::debug import foo.debug.*;</code></p></td>
<td><p>ASC 2.0 reports a syntax error if you don't use curly
braces. You need to enclose the code within curly braces as follows:</p>
<p><code>CONFIG::debug<br />
{<br />
import foo.debug.*;<br />
}</code></p></td>
<td><p>When you use a config block without curly
braces</p></td>
<td><p>Stricter</p></td>
</tr>
<tr>
<td><p>Vector only allows one type parameter</p></td>
<td><p>You can specify multiple type parameters, and the
extra ones would just be ignored.</p></td>
<td><p>A syntax error occurs.</p></td>
<td><p>When you specify multiple type of parameters for a
Vector:</p>
<p><code>var x:Vector.&lt;String, int&gt;;</code></p></td>
<td><p>Stricter</p></td>
</tr>
<tr>
<td><p>'<code>default</code>' cannot be used as an
identifier name</p></td>
<td><p>You can define code as follows:</p>
<p><code>var x = &lt;{default} /&gt;</code></p>
<p>However, a runtime error occurs.</p></td>
<td><p>The following syntax error occurs: "Expecting
identifier but found keyword '<code>default</code>'."</p></td>
<td><p>When you use the keyword '<code>default</code>' as an
identifier name embedded in XML literals.</p>
<p> </p></td>
<td><p>Stricter</p></td>
</tr>
<tr>
<td><p>Warn when local variable has the same name as
function argument</p></td>
<td><p>You do not receive a warning when you define code as
follows:</p>
<p><code>function bas(x:int)<br />
{<br />
 var x:int = x;<br />
}</code></p></td>
<td><p>The following warning appears: "Warning 5040:
variable definition duplicates function parameter: arg."</p></td>
<td><p>When you try to redefine a function argument:</p>
<p><code>function bas(x:int)<br />
{<br />
 var x:int = x;<br />
}</code></p></td>
<td>Stricter</td>
</tr>
<tr>
<td><p>Warn on duplicate definitions of local
functions</p></td>
<td><p>You do not receive a warning when you define code as
follows:</p>
<p><code>function foo()<br />
{<br />
 function baz() {}<br />
 function baz() {}<br />
}</code></p></td>
<td><p>The following warning appears: "Warning 1021 :
Duplicate function definition: baz"</p></td>
<td><p>When you define duplicate local function definitions
as follows:</p>
<p><code>function foo()<br />
{<br />
 function baz() {}<br />
 function baz() {}<br />
}</code></p></td>
<td><p>Stricter</p></td>
</tr>
<tr>
<td><p><code>public::</code> qualifier may now resolve to
multiple namespaces depending on the context</p></td>
<td><p>Defining <code>public::</code> in a package meant
public(package-name)</p></td>
<td>Defining <code>public::</code> in a package context now means
public(package-name), unnamed public</td>
<td><p>When you qualify an expression with
<code>public::</code></p></td>
<td><p>More lenient (some unresolved errors will now
resolve), and silent change in code generation</p></td>
</tr>
<tr>
<td><p>Type annotations that resolve to locals are detected
more consistently in ASC 2.0</p></td>
<td><p>The compiler would not always give an error if a type
annotation resolved to a local, such as in "<code>var String:String"</code>.</p>
<p>And in some cases, the compiler did not report an error, though it
should have.</p></td>
<td><p>ASC 2.0 will always issue the appropriate error if a
type annotation does not resolve to a type.</p></td>
<td><p>When you define code that collides with a type as
follows:<br />
<br />
<code>var String:String;</code></p></td>
<td><p>Stricter</p></td>
</tr>
<tr>
<td><p>Package directives must be first</p></td>
<td><p>Any package directives had to occur before any other
code in a file.</p></td>
<td>Packages can be declared anywhere in a file, as long as
they are declared in a global context (and not nested inside another
construct).</td>
<td><p>When you define a package declaration after some
other code.</p></td>
<td><p>More lenient</p></td>
</tr>
<tr>
<td><p>Improve constant evaluation</p></td>
<td><p>Not many constants were evaluated at compile
time.<br />
<br />
A reference like <code>MyClass.SOME_CONSTANT</code> would typically
cause MyClass to be statically initialized if it had not already
been.</p></td>
<td>Many more constants are evaluated at compile time.<br />
<br />
This could change initialization order of classes. Since references to
static constants may be replaced by the computed constant values,
<code>MyClass.SOME_CONSTANT</code> may not cause MyClass to be
statically initialized.</td>
<td><p>When referencing static constants and when depending
on classes to be initialized in a particular order.</p></td>
<td><p>Silent change in code generation</p></td>
</tr>
<tr>
<td><p>More consistent warnings for "scoped to default
namespace"</p></td>
<td><p>Only warned on classes and class methods that didn't
have namespace specs (like "<code>public</code>")</p></td>
<td><p>ASC 2.0 also warns on namespaces, functions, and
variables</p></td>
<td><p>Users will often see this with legacy code.</p></td>
<td><p>Stricter</p></td>
</tr>
<tr>
<td><p>Hoist local variables earlier</p></td>
<td><p>Local variables were hoisted as they were
encountered.<br />
<br />
Earlier identifiers might not resolve to the hoisted variable.</p></td>
<td>All local variables are hoisted before any identifiers
are resolved.<br />
<br />
An identifier in a function will always resolve to a local variable in
that function if such a local variable exists.</td>
<td><p>When referencing a local variable before its
declaration.</p></td>
<td><p>Silent change in code generation</p></td>
</tr>
<tr>
<td><p>Evaluate "<code>CONFIG const</code>" expressions
correctly</p></td>
<td><p>Initial-value expressions for config constants, like
the following, were often not evaluated correctly:</p>
<p><code>CONFIG const myConst = 10 || 0;</code></p></td>
<td><p>Initial-value expressions for config constants are
evaluated correctly, using the same rules as other expressions.</p></td>
<td><p>When using config constant expressions.</p></td>
<td><p>Silent change in code generation</p></td>
</tr>
<tr>
<td><p>Strip metadata in debug SWFs</p></td>
<td>Metadata was stripped in release SWF files, based on the
<code>-keep-as3-metadata</code> option, but not in debug SWF files.</td>
<td><p>Metadata is stripped in both debug and release SWF
files.</p></td>
<td><p>When you use
<code>-keep-as3-metadata</code>.</p></td>
<td><p>Silent change in code generation</p></td>
</tr>
<tr>
<td><p>Generate fewer classes for embedded assets</p></td>
<td>A class was generated for each variable with Embed(...)
metadata, even if multiple variables referred to the same embedded asset
tag.<br />
<br />
Embed variables were never equal because they referred to different
classes.</td>
<td><p>A class is generated for each embedded asset
tag.<br />
<br />
Embed variables may now be equal because they may refer to the same
class.</p></td>
<td><p>When you use embedded assets.</p></td>
<td><p>Silent change in code generation</p></td>
</tr>
<tr>
<td><p>Disallow Array and Object literals at the beginning
of a statement</p></td>
<td><p>Code like the following is compiled without
errors:</p>
<p><code>[1,2].length</code><br />
and<br />
<code>{"name":"hello"}.name</code></p>
<p> </p></td>
<td><p>You get a compiler error: "unbound metadata problem"
for:</p>
<ul class="incremental">
<li>Array literal</li>
<li>extra characters problem</li>
<li>for Object literal</li>
</ul></td>
<td><p>When you define Array literal or Object literal at
the beginning of a statement.</p>
<p><code>[1,2].length</code><br />
and<br />
<code>{"name":"hello"}.name</code></p></td>
<td>Stricter</td>
</tr>
<tr>
<td><p>Disallow a for-loop iterator const that conflicts
with an existing variable</p></td>
<td><p>Code like the following is compiled without
errors:</p>
<p><code>function<br />
{<br />
 var x;<br />
 for<br />
 (const x in [1, 2,3])<br />
   {<br />
   }<br />
 }</code></p></td>
<td><p>You get the following compiler error:</p>
<p>"<code>const x</code>" conflicts with "<code>var x</code>"</p></td>
<td><p>When re-declaring an existing variable as a const
loop iterator variable.</p>
<p><code>function<br />
{<br />
 var x;<br />
 for<br />
(const x in [1,2,3])<br />
   {<br />
   }<br />
}</code></p></td>
<td><p>Stricter</p></td>
</tr>
<tr>
<td><p>Disallow a local const that conflicts with a function
parameter</p></td>
<td><p>Code like the following is compiled without
errors:</p>
<p><code>function f(p:*)<br />
{<br />
 const p:*;<br />
}</code></p></td>
<td><p>A compiler error occurs.</p></td>
<td><p>When declaring a local const with the same name as a
parameter of the containing function.</p>
<p><code>function f(p:*)<br />
{<br />
 const p:*;<br />
}</code></p></td>
<td><p>Stricter</p></td>
</tr>
<tr>
<td><p>Disallow void as an expression</p></td>
<td><p>Code like the following is compiled without
errors:</p>
<p><code>foo(void)</code></p></td>
<td><p>The following compiler error occurs:</p>
<p>Syntax error: "void" is not allowed here.</p></td>
<td><p>When you use void as an expression instead of a type
annotation or an operator:</p>
<p><code>foo(void)</code></p></td>
<td><p>Stricter</p></td>
</tr>
<tr>
<td><p>Disallow a function definition in a 'with' statement
without a curly block</p></td>
<td><p>Code like the following is compiled without
errors:</p>
<p><code>with(...) function x() {}</code></p></td>
<td><p>The following compiler error occurs:</p>
<p>Syntax error: "function" is not allowed here.</p></td>
<td><p>When defining a function using a 'with' statement
without curly braces:</p>
<p><code>with(...) function x() {}</code></p></td>
<td>Stricter</td>
</tr>
<tr>
<td><p>Allow an anonymous function as an expression
statement</p></td>
<td><p>The following compiler error occurs:</p>
<p>Syntax error: Expecting identifier before "(".</p></td>
<td><p>No compiler error.</p></td>
<td><p>When defining an anonymous function without assigning
it to any variable.</p></td>
<td><p>More lenient</p></td>
</tr>
<tr>
<td><p>Evaluate the left-hand-side of compound assignments
only once</p></td>
<td><p>Code like:</p>
<p><code>onlyOnce().a += 1</code></p>
<p>would run<code> onlyOnce()</code> twice.</p></td>
<td><p>Runs <code>onlyOnce()</code> only once.</p></td>
<td><p>When you write compound assignments with
complex left-hand side values, which depend on whether parts of the
left-hand side values are evaluated once or twice.</p></td>
<td><p>Silent change in code generation</p></td>
</tr>
<tr>
<td><p>Process metadata on global functions</p></td>
<td><p>Code like:</p>
<p><code>[Foo]</code><br />
<code>function bar()</code></p>
<p>did not produce metadata for <code>bar()</code></p></td>
<td><p>Metadata is produced for <code>bar()</code>.</p></td>
<td><p>When writing metadata on global functions and
expecting it to be ignored.</p></td>
<td><p>Silent change in code generation</p></td>
</tr>
<tr>
<td><p>Always use "static mode" semantics</p></td>
<td><p>Static semantics were turned on by <code>-strict</code> flag.</p></td>
<td><p>Static semantics are always enabled.</p></td>
<td><p>Trait entries of global and nested functions are
allocated as const entries of type Function.</p>
<p>Names that can be resolved to a unique definition are written out as
QNames. This could be problematic if a program were compiled with one
version of a library and run with a different version of that
library.</p></td>
<td><p>Stricter</p></td>
</tr>
<tr>
<td><p>Allow multiple definitions with the same unqualified
name as long as they have disjoint namespaces</p></td>
<td><p>Code like:</p>
<p><code>public class Foo {}<br />
private class Foo {}</code></p>
<p>produces a compiler error.</p></td>
<td>No compiler error occurs.</td>
<td><p>When you declare multiple definitions with the same
unqualified name:</p>
<p><code>public class Foo {}<br />
private class Foo {}</code></p></td>
<td><p>Lenient</p></td>
</tr>
<tr>
<td><p>Report all ambiguous definitions</p></td>
<td>Only the second and subsequent ambiguous definition
sites were reported.</td>
<td><p>All ambiguous definition sites are reported.</p></td>
<td><p>When declaring multiple definitions with the same
unqualified name.</p></td>
<td><p>Diagnostic change</p></td>
</tr>
<tr>
<td><p>Disallow const / var conflicts</p></td>
<td><p>Code like:</p>
<p><code>var foo;<br />
const foo;</code></p>
<p>is compiled without errors. The var "overrode" the const.</p></td>
<td><p>A compiler error occurs. The const declaration
conflicts with var declaration independent of their order.</p></td>
<td><p>When you declare a const with the same name as a
var:</p>
<p><code>var foo;<br />
const foo;</code></p></td>
<td><p>Stricter</p></td>
</tr>
<tr>
<td><p>Disallow increment/decrement of const</p></td>
<td><p>Code like:</p>
<p><code>const i;<br />
i++;</code></p>
<p>is compiled without errors.</p></td>
<td><p>The following compiler error occurs:</p>
<p>"Increment operand is invalid."</p></td>
<td><p>When you try to increment or decrement a const:</p>
<p><code>const i;<br />
i++;</code></p></td>
<td><p>Stricter</p></td>
</tr>
<tr>
<td><p>Prevent leakage of packages from import statements
into other scopes</p></td>
<td>While import p.C, p was interpreted as a package later
in the file, even outside the scope where the import statement
occurred.</td>
<td><p>p is interpreted as a package only within the scope
where the import occurs.</p></td>
<td><p>When you use a fully-qualified type name without an
import statement within scope.</p></td>
<td><p>Stricter</p></td>
</tr>
<tr>
<td><p>Disallow duplicate fully qualified names in the
source path</p></td>
<td>No compiler error occurred when there were duplicate
names in the source path. However, if the duplicate name is referenced,
a "Could not resolve X to a component implementation" error is
reported.</td>
<td><p>The following compiler error occurs:</p>
<p>Error: X is defined by multiple files: src/X.mxml, src/X.as</p>
<p> </p></td>
<td><p>When setting the source path of the project.</p></td>
<td><p>Stricter</p></td>
</tr>
<tr>
<td><p>Disallow reassignment of a named function</p></td>
<td><p>A named function could be re-assigned a different
value.</p></td>
<td><p>The following runtime error occurs:</p>
<p>"Error #1074: Illegal write to read-only property factorial on
&lt;anonymous&gt;."</p></td>
<td><p>When trying to reassign the value of a named
function.</p></td>
<td><p>Stricter</p></td>
</tr>
<tr>
<td><p>Disallow metadata that requires runtime
evaluation</p></td>
<td>Metadata that contained expressions that would need to
be evaluated at runtime was completely ignored by the compiler.</td>
<td><p>A compiler error occurs.</p>
<p>A syntax error is generated for metadata constructs that contain
function calls or other expressions that do not match the metadata
syntax.</p></td>
<td><p>When writing metadata that requires runtime
evaluation.</p></td>
<td><p>Stricter</p></td>
</tr>
<tr>
<td><p>Disallow invalid Vector assignment</p></td>
<td>The compiler did not catch invalid vector type
assignment, such as assigning Vector.&lt;int&gt; to Vector.&lt;uint&gt;
or Vector.&lt;Number&gt; to Vector.&lt;int&gt;. This would fail at
runtime.</td>
<td><p>A compiler error occurs.</p></td>
<td><p>When assigning one Vector type to another Vector
type.</p></td>
<td><p>Stricter</p></td>
</tr>
<tr>
<td><p>Allow null to be divided by any value</p></td>
<td><p>Code like:</p>
<p><code>x = null / 2;</code></p>
<p>produced the compiler error:<br />
"Error #1084: Syntax error"</p></td>
<td><p>No compiler error occurs.</p></td>
<td><p>When attempting to divide null by anything.</p></td>
<td><p>More lenient</p></td>
</tr>
<tr>
<td><p>Flow-aware analysis looks inside tautological if/else
alternatives</p></td>
<td><p>The legacy compiler was "slightly" flow-aware when it
analyzed code; problems in tautologies were ignored.</p></td>
<td><p>A non-flow-aware analysis finds problems in
tautologies.</p></td>
<td><p>When you define code like the following:</p>
<p><code>function AClassConstructor() {<br />
super();<br />
if ( false )<br />
super();  // ASC 2.0 reports problems, legacy compiler does not<br />
}</code></p>
<p><code>function f():int<br />
{<br />
if ( false )<br />
return;  // ASC 2.0 reports problems, legacy compiler does not<br />
return 44;<br />
}</code></p></td>
<td><p>Stricter</p></td>
</tr>
<tr>
<td><p>Disallow classes nested within try, catch, and
finally</p></td>
<td><p>The legacy compiler allowed definitions in
package-level and file-level try and finally blocks.</p></td>
<td><p>A compiler error occurs.</p>
<p>A syntax error occurs when a class is encountered in a try, catch, or
finally block.</p></td>
<td><p>When attempting to declare a class nested within
a try, catch, or finally block.</p></td>
<td><p>Stricter</p></td>
</tr>
<tr>
<td><p>Warn on assignment in any conditional
expression</p></td>
<td><p>The legacy compiler only warns for assignment as an
immediate child of an if.</p></td>
<td>A warning appears if an assignment is used (alone or via
Boolean operations) as part of a conditional expression.  However, no
warning appears if an assignment is part of an arithmetic or comparison
expression.</td>
<td><p>When attempting to use an assignment expression in a
conditional, for example, while (x = 3).</p></td>
<td><p>Stricter</p></td>
</tr>
<tr>
<td><p>Detect ambiguous base class names caused by
import</p></td>
<td><p>The legacy compiler resolved names inside packages as
if the package was a defining-scope but it's a weaker
opening-scope.</p></td>
<td><p>The compiler identifies ambiguous references caused
by importing other package names:</p>
<p>"Error: Ambiguous reference to A<br />
public class B extends A {}"</p></td>
<td><p>When you define code, like the following, where Base
class A is ambiguous:</p>
<p><code>package P1<br />
{<br />
 public class A{}<br />
}</code></p>
<p><code>package P2<br />
{<br />
 public class A {}<br />
 class B extends A {}<br />
}<br />
import P1.*;</code></p></td>
<td><p>Stricter</p></td>
</tr>
<tr>
<td><p>Initialize constants in locals at their hoist
point</p></td>
<td><p>Legacy compiler initialized constants at the point
where they were declared.</p></td>
<td><p>The compiler initializes the constant at its
definition.</p></td>
<td><p>When you use such a constant: the code emits 5, the
legacy compiler emits 3:</p>
<p><code>function test()<br />
{<br />
var accum = constInMiniscope;<br />
 for<br />
(var i = 0; i &lt; 2; i++)<br />
  {<br />
   accum += constInMiniscope;<br />
    const constInMiniscope = 1;<br />
    accum +=  constInMiniscope;<br />
  }<br />
 return accum;<br />
}</code></p></td>
<td><p>Silent change in code generation</p></td>
</tr>
<tr>
<td><p>"<code>goto</code>" is a keyword</p></td>
<td><p>Functions, variables, and classes could be named
"<code>goto</code>"</p></td>
<td><p>"<code>goto</code>" is a keyword in ASC 2.0, and thus
cannot be used as the name of a definition.</p></td>
<td>While compiling AS3 code that uses <code>"goto"
</code>as the name of a definition.</td>
<td><p> </p></td>
</tr>
<tr>
<td><p>Warn about unbound metadata</p></td>
<td>The legacy compiler silently ignored metadata that
wasn't immediately above a definition.</td>
<td><p>ASC 2.0 will ignore such metadata but report a
warning.</p></td>
<td><p>When writing metadata.</p></td>
<td><p>Stricter</p></td>
</tr>
</tbody>
</table>

### Flash Builder Changes

<table>
<colgroup>
<col style="width: 20%" />
<col style="width: 20%" />
<col style="width: 20%" />
<col style="width: 20%" />
<col style="width: 20%" />
</colgroup>
<tbody>
<tr>
<th>Issue</th>
<th>Behavior in legacy compiler</th>
<th>Behavior in ASC 2.0</th>
<th>When will a user encounter this issue?</th>
<th>Severity of change</th>
</tr>
<tr>
<td><p>Path resolution rules have changed</p></td>
<td><p>Inconsistent logic for resolving paths.</p></td>
<td><p>The compiler now has a defined path resolution behavior.</p>
<ul class="incremental">
<li>Paths in configuration options are now consistently resolved.</li>
<li>Only configuration options can leverage Flash Builder linked
resources and named project paths, these are no longer honored in source
code.</li>
<li>Paths in source code (for example, AS3 includes, MXML sources, embed
statements) are now consistently resolved.</li>
<li>Relative paths are resolved relative to the including file.</li>
<li><p>To refer to sources and assets relative to a source root (or for
assets relative to a library root, for example, assets inside a SWC
file), use a forward slash '/'.<br />
</p>
<em>Note: On Mac OSX, a forward slash will first be checked for a fully-qualified file path.</em></li>
<li>Sources and assets are now consistently resolved, with the source
path searched first. Assets are only searched for in the library-path
last.</li>
</ul></td>
<td><p>When using relative paths in configuration, embedded assets,
source includes.</p></td>
<td><p>Stricter</p></td>
</tr>
<tr>
<td><p>Disallow more ambiguous references</p></td>
<td><p>The old mxmlc and compc did not find certain ambiguous references
that the legacy ActionScript compiler found.</p></td>
<td><p>Ambiguous references are correctly found.</p></td>
<td><p>For example, when you declare protected foo and ns foo and do use
namespace ns and refer to just foo.</p></td>
<td><p>Stricter</p></td>
</tr>
<tr>
<td><p>Disallow URIs for embedded assets</p></td>
<td><p>Code like:</p>
<p><code>Embed(source="http://...")</code></p>
<p>compiles without errors, downloading and embedding the remote
asset.</p></td>
<td><p>A compiler error occurs.</p>
<p> </p>
<p>The compiler no longer supports URIs as source locations.
<code>http://</code> and <code>file://</code> produce an error
saying that the source could not be found.</p></td>
<td><p>When you use an URI to specify an embedded asset.</p></td>
<td><p>Stricter</p></td>
</tr>
<tr>
<td><p>Change the compc error message when looking for an include-class
in the source-path that is not defined</p></td>
<td><p>Code like:</p>
<p><code>foo/bar.as:</code></p>
<p>produces the error: "A file found in a source-path must have an
externally visible definition. If a definition in the file is meant to
be externally visible, please put the definition in a package."</p></td>
<td><p>The following error occurs: Could not find source for class
foo.bar.</p></td>
<td><p>When you use compc to compile a class with no externally visible
definitions:<br />
<br />
<code>foo/bar.as = var x:String = "x";<br />
fcompc<br />
-load-config=...<br />
-compiler.source-path foo<br />
-include-classes foo.bar<br />
-output out.swc</code> </p></td>
<td><p>Diagnostic change</p></td>
</tr>
<tr>
<td><p>No longer report the optimized ABC size in link reports</p></td>
<td><p>Generated an optimized size entry in SWF link reports.</p></td>
<td><p>The entry is removed.</p></td>
<td><p>When you use -link-report.</p></td>
<td><p>N/A</p></td>
</tr>
</tbody>
</table>
