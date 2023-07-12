---
description: XML External Entity (XXE) Injection
---

# XXE

## Intro

Occur when XML data is taken from a user-controlled input without properly sanitizing or safely parsing it, which may allow us to use XML features to perform malicious actions.

* can cause considerable damage to a web app and back-end
* disclosing sensitive files to shutdown the back-end server
* one of the [Top 10 Web Security Risks](https://owasp.org/www-project-top-ten/) by OWASP

## XML

* is a markup language like HTML and SGML
* designed for flexible transfer and storage of data and documents
* **Not** focused on displaying data but mostly on storing documents' data
* XML documents are formed of element trees
* each element is denoted by a `tag`
* first element is called the `root element`, while other elements are `child elements`.

**E.G.**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<email>
  <date>01-01-2022</date>
  <time>10:00 am UTC</time>
  <sender>john@inlanefreight.com</sender>
  <recipients>
    <to>HR@inlanefreight.com</to>
    <cc>
        <to>billing@inlanefreight.com</to>
        <to>payslips@inlanefreight.com</to>
    </cc>
  </recipients>
  <body>
  Hello,
      Kindly share with me the invoice for the payment made on January 1, 2022.
  Regards,
  John
  </body> 
</email>
```

The above example shows some of the key elements of an XML document, like:

<table><thead><tr><th width="162.33333333333331">Key</th><th width="410">Definition</th><th>Example</th></tr></thead><tbody><tr><td><code>Tag</code></td><td>The keys of an XML document, usually wrapped with (<code>&#x3C;</code>/<code>></code>) characters.</td><td><code>&#x3C;date></code></td></tr><tr><td><code>Entity</code></td><td>XML variables, usually wrapped with (<code>&#x26;</code>/<code>;</code>) characters.</td><td><code>&#x26;lt;</code></td></tr><tr><td><code>Element</code></td><td>The root element or any of its child elements, and its value is stored in between a start-tag and an end-tag.</td><td><code>&#x3C;date>01-01-2022&#x3C;/date></code></td></tr><tr><td><code>Attribute</code></td><td>Optional specifications for any element that are stored in the tags, which may be used by the XML parser.</td><td><code>version="1.0"</code>/<code>encoding="UTF-8"</code></td></tr><tr><td><code>Declaration</code></td><td>Usually the first line of an XML document, and defines the XML version and encoding to use when parsing it.</td><td><code>&#x3C;?xml version="1.0" encoding="UTF-8"?></code></td></tr></tbody></table>

some characters are used as part of an XML document structure, like `<`, `>`, `&`, or `"`.

if we need to use them in an XML document, we should replace them with entity references (e.g. `&lt;`, `&gt;`, `&amp;`, `&quot;`).

## XML DTD

* `XML Document Type Definition (DTD)`
* allows the validation of an XML document against a pre-defined document structure.
* pre-defined document structure can be defined in the document itself or in an external file

```xml
<!DOCTYPE email [
  <!ELEMENT email (date, time, sender, recipients, body)>
  <!ELEMENT recipients (to, cc?)>
  <!ELEMENT cc (to*)>
  <!ELEMENT date (#PCDATA)>
  <!ELEMENT time (#PCDATA)>
  <!ELEMENT sender (#PCDATA)>
  <!ELEMENT to  (#PCDATA)>
  <!ELEMENT body (#PCDATA)>
]>
```

Above example,&#x20;

* `DTD` declare root email element with `ELEMENT` type and denoting child elements
* Each child is declared (some have their own child elements, some only have raw data)

The `DTD` can be placed in XML document rith after XML Declaration in first line.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email SYSTEM "email.dtd">
```

It is also possible to reference a DTD through a URL, as follows:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email SYSTEM "http://inlanefreight.com/email.dtd">
```

similar to how HTML documents define and reference JavaScript and CSS scripts.

## XML Entities

* custom entities -> XML variables
* can be used to store repetitive data
* use `ENITITY` keyword followed by entity name and it's value

```
<!ENTITY company "Inlane Freight">
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [
  <!ENTITY company "Inlane Freight">
]>
```

* To reference entity in XML document use `&` and `;` (e.g. `&company;`)
* After referencing, it will be replaced with it's value by XML parser
* we can `reference External XML Entities` with the `SYSTEM` keyword, which is followed by the external entity's path

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [
  <!ENTITY company SYSTEM "http://localhost/company.txt">
  <!ENTITY signature SYSTEM "file:///var/www/html/signature.txt">
]>
```

* We may use `PUBLIC` instead of `SYSTEM` for loading external resource
* `PUBLIC` is used with publicly declared entities and standards (`lang="en"`)

<mark style="color:red;">**When the XML file is parsed on the server-side, in cases like SOAP (XML) APIs or web forms, then an entity can reference a file stored on the back-end server, which may eventually be disclosed to us when we reference the entity.**</mark>
