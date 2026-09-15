# ERB to EPP Converter

A CLI/web tool written in rust that converts embedded ruby (.erb) templates into embedded puppet (.epp) compatible syntax by:
 - Replacing @ with $ inside template tags
 - Converting if, unless, elsif, else, and end into proper .epp blocks with curly brackets
 - Converting simple .each loops into proper .epp
 - Converting complex .each loops that iterate through empty hashes (with || {}) by adding an if
 - Preserving optional whitespace trimmers (<%- and -%>)
 - Leaving @ outside tags (like in email addresses) unchanged
 - Converting versioncmp fn into proper .epp
 - Add missing $ to variables inside tags
 - Add parameter tags like <%- | Hash $hash | -%> to beginning of new epp file

### 🌐 Web Interface (New)
Run this converter directly in the browser with WebAssembly.

1. Install dependencies and build the Wasm package to `pkg/`:
   ```bash
   cargo install wasm-pack
   wasm-pack build --target web
   ```
2. Start a local webserver:
   ```bash
   python3 -m http.server 8000
   # or
   ruby -run -e httpd . -p 8000
   ```
3. Open http://localhost:8000

**Coming soon: Website**

### CLI Usage

```
#convert and print to stdout
./erb-to-epp input.erb
```
```
#convert and save to an output file
./erb-to-epp input.erb output.epp
```

### Example

input.erb
```
<%- if @name -%>
  Hello <%= @name %>
<% elsif @other_name %>
  Hello <%= @other_name %>
<%- else -%>
  No variable
<%- end -%>

<%- unless $name -%>
  No Name
<%- end -%>

<% @features.each do | f | %>
  <%= f %>
<% end %>
<%- @features.each do | f, g, h | -%>
  <%= f %><%= g['something'] %><%= h -%>
<% end %>

<% if scope.function_versioncmp([@version, '1.0']) < 0 %>
  Do Something
<% end %>

<%- (@vars['thing1']['thing2']['thing3'] || {}).each do | var | -%>
    Thing =<%= var['thing'] %>
<% end -%>

An Email: test@gmx.de
```
output.epp
```
<%- | Hash $features,
      String $name,
      String $other_name,
      Array $vars,
      String $version,
| -%>
<%- if $name { -%>
  Hello <%= $name %>
<% } else if $other_name { %>
  Hello <%= $other_name %>
<%- } else { -%>
  No variable
<%- } -%>

<%- unless $name { -%>
  No Name
<%- } -%>

<% $features.each | $f | { %>
  <%= $f %>
<% } %>
<%- $features.each | $f, $g, $h | { -%>
  <%= $f %><%= $g['something'] %><%= $h -%>
<% } %>

<% if versioncmp($version, '1.0') < 0 { %>
  Do Something
<% } %>

<%- if ('thing1' in $vars) and ('thing2' in $vars['thing1']) and ('thing3' in $vars['thing1']['thing2']) and ($vars['thing1']['thing2']['thing3'] =~ Array) { -%>
<%- $vars['thing1']['thing2']['thing3'].each | $var | { -%>
    Thing =<%= $var['thing'] %>
<% } -%>
<%- } -%>

An Email: test@gmx.de
```
