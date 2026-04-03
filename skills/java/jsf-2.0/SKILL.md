---
name: jsf-2.0
description: >
  JSF 2.0 (JavaServer Faces) patterns, configuration, and constraints for Java EE 6 web applications.
  Trigger: When working in a JSF 2.0 codebase, when Facelets XHTML views are present, when faces-config.xml exists.
metadata:
  author: paulschick
  version: "1.0"
---

## When to Use

Load this skill when:

- Project contains `faces-config.xml`
- `web.xml` declares `javax.faces.webapp.FacesServlet`
- Source contains `.xhtml` Facelets view files
- Java source imports `javax.faces.*` packages
- Project targets Java EE 6 with JSF 2.0 (JSR 314)

## Configuration

### web.xml

Declare `FacesServlet` and map it to `*.xhtml`. Set `PROJECT_STAGE` to `Development` during development for
better error messages.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="3.0"
         xmlns="http://java.sun.com/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
        http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd">

    <context-param>
        <param-name>javax.faces.PROJECT_STAGE</param-name>
        <param-value>Development</param-value>
    </context-param>

    <servlet>
        <servlet-name>Faces Servlet</servlet-name>
        <servlet-class>javax.faces.webapp.FacesServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>Faces Servlet</servlet-name>
        <url-pattern>*.xhtml</url-pattern>
    </servlet-mapping>

    <welcome-file-list>
        <welcome-file>index.xhtml</welcome-file>
    </welcome-file-list>
</web-app>
```

### faces-config.xml

Minimal `faces-config.xml` for JSF 2.0. Most configuration is handled by annotations; use this file for
navigation rules and registrations that cannot be annotation-based.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<faces-config xmlns="http://java.sun.com/xml/ns/javaee"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
        http://java.sun.com/xml/ns/javaee/web-facesconfig_2_0.xsd"
              version="2.0">

    <navigation-rule>
        <from-view-id>/login.xhtml</from-view-id>
        <navigation-case>
            <from-outcome>success</from-outcome>
            <to-view-id>/dashboard.xhtml</to-view-id>
            <redirect/>
        </navigation-case>
    </navigation-rule>
</faces-config>
```

## Critical Patterns

### Pattern 1: Managed beans with CDI

Prefer `@Named` (from `javax.inject`) with CDI scope annotations over JSF-native `@ManagedBean`. CDI provides
better dependency injection, interceptors, and lifecycle management.

```java
import java.io.Serializable;
import javax.inject.Named;
import javax.enterprise.context.SessionScoped;

@Named
@SessionScoped
public class UserBean implements Serializable {

    private static final long serialVersionUID = 1L;
    private String username;
    private String email;

    public String login() {
        // authentication logic
        return "dashboard?faces-redirect=true";
    }

    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
}
```

Legacy alternative (avoid in new code):

```java
import javax.faces.bean.ManagedBean;
import javax.faces.bean.SessionScoped;

@ManagedBean
@SessionScoped
public class UserBean implements Serializable { ... }
```

### Pattern 2: Bean scopes

Choose the narrowest scope that satisfies the requirement.

| Scope       | Annotation           | Survives            | Use for                               |
|-------------|----------------------|---------------------|---------------------------------------|
| Request     | `@RequestScoped`     | Single request      | Form processing, stateless operations |
| View        | `@ViewScoped`        | Same view postbacks | Ajax-heavy pages, multi-step forms    |
| Session     | `@SessionScoped`     | Browser session     | Login state, user preferences         |
| Application | `@ApplicationScoped` | App lifetime        | Shared lookup data, caches            |

Default to `@RequestScoped`. Escalate to `@ViewScoped` when data must survive Ajax postbacks to the same page.
Use `@SessionScoped` sparingly — only for data that genuinely belongs to the user session.

```java
import javax.inject.Named;
import javax.faces.view.ViewScoped;
import java.io.Serializable;
import java.util.List;

@Named
@ViewScoped
public class SearchBean implements Serializable {

    private static final long serialVersionUID = 1L;
    private String query;
    private List<Result> results;

    public void search() {
        results = searchService.find(query);
    }

    public String getQuery() { return query; }
    public void setQuery(String query) { this.query = query; }
    public List<Result> getResults() { return results; }
}
```

### Pattern 3: Facelets templating

Use `ui:composition` with a template for consistent page layout. Define insertable sections with `ui:insert`
in the template and fill them with `ui:define` in pages.

Template (`WEB-INF/templates/layout.xhtml`):

```xml
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://java.sun.com/jsf/html"
      xmlns:ui="http://java.sun.com/jsf/facelets">
    <h:head>
        <title>
            <ui:insert name="title">Default Title</ui:insert>
        </title>
        <h:outputStylesheet library="css" name="style.css"/>
    </h:head>
    <h:body>
        <div id="header">
            <ui:insert name="header">
                <ui:include src="/WEB-INF/includes/header.xhtml"/>
            </ui:insert>
        </div>
        <div id="content">
            <ui:insert name="content">Default content</ui:insert>
        </div>
        <div id="footer">
            <ui:insert name="footer">
                <ui:include src="/WEB-INF/includes/footer.xhtml"/>
            </ui:insert>
        </div>
    </h:body>
</html>
```

Page using the template:

```xml

<ui:composition xmlns="http://www.w3.org/1999/xhtml"
                xmlns:h="http://java.sun.com/jsf/html"
                xmlns:ui="http://java.sun.com/jsf/facelets"
                template="/WEB-INF/templates/layout.xhtml">

    <ui:define name="title">User Profile</ui:define>

    <ui:define name="content">
        <h:form>
            <h:outputLabel for="name" value="Name:"/>
            <h:inputText id="name" value="#{userBean.username}"/>
            <h:commandButton value="Save" action="#{userBean.save}"/>
        </h:form>
    </ui:define>
</ui:composition>
```

### Pattern 4: Form handling

Wrap input components in `h:form`. Bind values to bean properties with EL expressions. Trigger action methods
with `h:commandButton`.

```xml

<h:form id="registrationForm">
    <h:panelGrid columns="2">
        <h:outputLabel for="name" value="Name:"/>
        <h:inputText id="name" value="#{registrationBean.name}"
                     required="true"
                     requiredMessage="Name is required"/>

        <h:outputLabel for="email" value="Email:"/>
        <h:inputText id="email" value="#{registrationBean.email}"
                     required="true"
                     requiredMessage="Email is required">
            <f:validateRegex pattern="[^@]+@[^@]+\.[^@]+"/>
        </h:inputText>

        <h:outputLabel for="age" value="Age:"/>
        <h:inputText id="age" value="#{registrationBean.age}">
            <f:validateLongRange minimum="18" maximum="120"/>
        </h:inputText>
    </h:panelGrid>

    <h:commandButton value="Register" action="#{registrationBean.register}"/>
    <h:messages globalOnly="true" styleClass="errors"/>
</h:form>
```

### Pattern 5: Navigation

**Implicit navigation:** Return the view name from an action method. JSF resolves it to `<name>.xhtml`.

```java
public String register() {
    // process registration
    return "confirmation"; // navigates to confirmation.xhtml
}
```

**Redirect:** Append `?faces-redirect=true` to prevent the POST-redirect-GET problem.

```java
public String login() {
    if (authenticate(username, password)) {
        return "dashboard?faces-redirect=true";
    }
    return null; // re-render current page
}
```

**Return null** to stay on the current page (re-renders with current state).

**Explicit navigation rules** in `faces-config.xml` when implicit navigation is insufficient:

```xml

<navigation-rule>
    <from-view-id>/checkout.xhtml</from-view-id>
    <navigation-case>
        <from-outcome>payment</from-outcome>
        <to-view-id>/payment.xhtml</to-view-id>
        <redirect/>
    </navigation-case>
    <navigation-case>
        <from-outcome>cancel</from-outcome>
        <to-view-id>/cart.xhtml</to-view-id>
        <redirect/>
    </navigation-case>
</navigation-rule>
```

### Pattern 6: Converters and validators

**Built-in converters:**

```xml

<h:inputText id="price" value="#{productBean.price}">
    <f:convertNumber type="currency" currencySymbol="$"/>
</h:inputText>

<h:inputText id="birthDate" value="#{userBean.birthDate}">
<f:convertDateTime pattern="yyyy-MM-dd"/>
</h:inputText>
```

**Built-in validators:**

```xml

<h:inputText id="zipCode" value="#{addressBean.zipCode}">
    <f:validateLength minimum="5" maximum="10"/>
</h:inputText>
```

**Custom converter:**

```java
import javax.faces.component.UIComponent;
import javax.faces.context.FacesContext;
import javax.faces.convert.Converter;
import javax.faces.convert.FacesConverter;

@FacesConverter("phoneConverter")
public class PhoneConverter implements Converter {

    @Override
    public Object getAsObject(FacesContext context, UIComponent component, String value) {
        if (value == null || value.trim().isEmpty()) {
            return null;
        }
        return value.replaceAll("[^0-9]", "");
    }

    @Override
    public String getAsString(FacesContext context, UIComponent component, Object value) {
        if (value == null) {
            return "";
        }
        String phone = value.toString();
        if (phone.length() == 10) {
            return String.format("(%s) %s-%s",
                phone.substring(0, 3), phone.substring(3, 6), phone.substring(6));
        }
        return phone;
    }
}
```

Usage: `<h:inputText converter="phoneConverter" value="#{contactBean.phone}"/>`

**Custom validator:**

```java
import javax.faces.application.FacesMessage;
import javax.faces.component.UIComponent;
import javax.faces.context.FacesContext;
import javax.faces.validator.FacesValidator;
import javax.faces.validator.Validator;
import javax.faces.validator.ValidatorException;

@FacesValidator("emailValidator")
public class EmailValidator implements Validator {

    @Override
    public void validate(FacesContext context, UIComponent component, Object value)
            throws ValidatorException {
        String email = (String) value;
        if (email != null && !email.matches("[^@]+@[^@]+\\.[^@]+")) {
            throw new ValidatorException(
                new FacesMessage(FacesMessage.SEVERITY_ERROR, "Invalid email format", null));
        }
    }
}
```

Usage: `<h:inputText validator="emailValidator" value="#{userBean.email}"/>`

### Pattern 7: Ajax with f:ajax

Nest `f:ajax` inside a component to enable partial page processing and rendering.

```xml

<h:form>
    <h:inputText id="search" value="#{searchBean.query}">
        <f:ajax event="keyup" execute="@this" render="results"
                listener="#{searchBean.search}"/>
    </h:inputText>

    <h:panelGroup id="results">
        <ui:repeat value="#{searchBean.results}" var="item">
            <div>#{item.name}</div>
        </ui:repeat>
    </h:panelGroup>
</h:form>
```

Key attributes:

| Attribute  | Purpose                             | Default                                                                  |
|------------|-------------------------------------|--------------------------------------------------------------------------|
| `execute`  | Components to process on server     | `@this`                                                                  |
| `render`   | Components to re-render in response | `@none`                                                                  |
| `event`    | DOM event that triggers the request | Component default (e.g., `valueChange` for inputs, `action` for buttons) |
| `listener` | Server-side method to invoke        | None                                                                     |

Special keywords for `execute` and `render`: `@this`, `@form`, `@all`, `@none`, or specific component IDs.

```xml
<!-- Ajax submit: process entire form, re-render results and messages -->
<h:commandButton value="Submit" action="#{bean.submit}">
    <f:ajax execute="@form" render="results messages"/>
</h:commandButton>
```

### Pattern 8: Lifecycle phases

JSF processes each request through six phases:

1. **Restore View** — rebuilds the component tree for the requested page
2. **Apply Request Values** — populates components with submitted form data; conversion happens here
3. **Process Validations** — runs validators on converted values
4. **Update Model Values** — pushes validated values into backing bean properties
5. **Invoke Application** — executes action methods and navigation
6. **Render Response** — renders the component tree to HTML

**`immediate="true"`** on a command component skips phases 3-4 (validation and model update). Use for cancel
buttons or actions that don't need validated input.

```xml
<!-- Cancel button: skips validation -->
<h:commandButton value="Cancel" action="#{wizard.cancel}" immediate="true"/>
```

**`immediate="true"`** on an input component causes its value to be processed during Apply Request Values
instead of Process Validations. Useful for inputs that control page flow.

### Pattern 9: Messages

Add messages from backing beans using `FacesContext`:

```java
import javax.faces.application.FacesMessage;
import javax.faces.context.FacesContext;

public String save() {
    try {
        service.save(entity);
        FacesContext.getCurrentInstance().addMessage(null,
            new FacesMessage(FacesMessage.SEVERITY_INFO, "Saved successfully", null));
        return null;
    } catch (Exception e) {
        FacesContext.getCurrentInstance().addMessage(null,
            new FacesMessage(FacesMessage.SEVERITY_ERROR, "Save failed: " + e.getMessage(), null));
        return null;
    }
}
```

Display messages in the view:

```xml
<!-- Global messages (clientId is null) -->
<h:messages globalOnly="true" styleClass="messages"/>

        <!-- Component-specific message -->
<h:inputText id="email" value="#{bean.email}" required="true"/>
<h:message for="email" styleClass="error"/>
```

Severity levels: `SEVERITY_INFO`, `SEVERITY_WARN`, `SEVERITY_ERROR`, `SEVERITY_FATAL`.

### Pattern 10: Resource handling

Place static resources under `resources/` in the web root. Reference them with JSF resource tags.

Directory structure:

```
webapp/
  resources/
    css/
      style.css
    js/
      app.js
    images/
      logo.png
```

```xml
<!-- In <h:head> or anywhere in the page -->
<h:outputStylesheet library="css" name="style.css"/>
<h:outputScript library="js" name="app.js" target="head"/>

        <!-- Image -->
<h:graphicImage library="images" name="logo.png" alt="Logo"/>
```

The `library` attribute maps to a subdirectory under `resources/`. The `name` attribute is the filename.
JSF handles versioning and caching automatically.

## Anti-Patterns

### Don't: Use JSP as the view technology

```xml
<!-- BAD — JSP is deprecated in JSF 2.0 -->
<%@ taglib uri="http://java.sun.com/jsf/html" prefix="h" %>
<h:form>
<h:inputText value="#{bean.name}"/>
</h:form>
```

```xml
<!-- GOOD — Facelets (.xhtml) -->
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://java.sun.com/jsf/html">
    <h:form>
        <h:inputText value="#{bean.name}"/>
    </h:form>
</html>
```

### Don't: Define managed beans in faces-config.xml

```xml
<!-- BAD — verbose, error-prone, hard to maintain -->
<managed-bean>
    <managed-bean-name>userBean</managed-bean-name>
    <managed-bean-class>com.example.UserBean</managed-bean-class>
    <managed-bean-scope>session</managed-bean-scope>
</managed-bean>
```

```java
// GOOD — annotation-based with CDI
@Named
@SessionScoped
public class UserBean implements Serializable { ... }
```

### Don't: Use @ManagedBean when CDI is available

```java
// BAD — JSF-native annotations, limited DI
import javax.faces.bean.ManagedBean;
import javax.faces.bean.ManagedProperty;
import javax.faces.bean.RequestScoped;

@ManagedBean
@RequestScoped
public class OrderBean {
    @ManagedProperty("#{cartBean}")
    private CartBean cart;
}
```

```java
// GOOD — CDI provides full dependency injection
import javax.inject.Named;
import javax.inject.Inject;
import javax.enterprise.context.RequestScoped;

@Named
@RequestScoped
public class OrderBean {
    @Inject
    private CartBean cart;
}
```

### Don't: Put business logic in action methods

```java
// BAD — backing bean doing business logic directly
@Named
@RequestScoped
public class OrderBean {

    public String placeOrder() {
        // validation, pricing, inventory check, payment, email...
        if (cart.getItems().isEmpty()) { return null; }
        double total = 0;
        for (Item item : cart.getItems()) {
            total += item.getPrice() * item.getQuantity();
            inventory.decrement(item.getSku(), item.getQuantity());
        }
        payment.charge(total);
        emailService.sendConfirmation(order);
        return "confirmation?faces-redirect=true";
    }
}
```

```java
// GOOD — delegate to service layer
@Named
@RequestScoped
public class OrderBean {

    @Inject
    private OrderService orderService;

    public String placeOrder() {
        orderService.place(cart);
        return "confirmation?faces-redirect=true";
    }
}
```

### Don't: Overuse @SessionScoped

```java
// BAD — search results don't belong in the session
@Named
@SessionScoped
public class SearchBean implements Serializable {
    private List<Product> results; // stays in memory for entire session
}
```

```java
// GOOD — ViewScoped: data lives only while the user is on this page
@Named
@ViewScoped
public class SearchBean implements Serializable {
    private List<Product> results; // cleared when user navigates away
}
```

## Quick Reference

| Task                     | Pattern                                                       |
|--------------------------|---------------------------------------------------------------|
| Declare a managed bean   | `@Named @RequestScoped public class Bean { ... }`             |
| Inject a dependency      | `@Inject private Service service;`                            |
| Bind input to bean       | `<h:inputText value="#{bean.property}"/>`                     |
| Submit a form            | `<h:commandButton value="Go" action="#{bean.method}"/>`       |
| Navigate to page         | Return `"pageName"` from action method                        |
| Redirect after action    | Return `"pageName?faces-redirect=true"`                       |
| Stay on current page     | Return `null` from action method                              |
| Ajax partial update      | `<f:ajax execute="@form" render="resultPanel"/>`              |
| Show validation error    | `<h:message for="componentId"/>`                              |
| Show global messages     | `<h:messages globalOnly="true"/>`                             |
| Add message from bean    | `FacesContext.getCurrentInstance().addMessage(null, msg)`     |
| Convert a date           | `<f:convertDateTime pattern="yyyy-MM-dd"/>`                   |
| Validate range           | `<f:validateLongRange minimum="1" maximum="100"/>`            |
| Page template            | `<ui:composition template="/WEB-INF/templates/layout.xhtml">` |
| Template section         | `<ui:define name="content">...</ui:define>`                   |
| Include fragment         | `<ui:include src="/WEB-INF/includes/header.xhtml"/>`          |
| Skip validation (cancel) | `<h:commandButton immediate="true" action="#{bean.cancel}"/>` |
| Load CSS                 | `<h:outputStylesheet library="css" name="style.css"/>`        |
| Load JS                  | `<h:outputScript library="js" name="app.js" target="head"/>`  |

## Resources

- JSF 2.0 Specification (JSR 314): https://jcp.org/en/jsr/detail?id=314
- javax.faces Javadoc: https://docs.oracle.com/javaee/6/api/javax/faces/package-summary.html
- Java EE 6 Tutorial (JSF chapter): https://docs.oracle.com/javaee/6/tutorial/doc/bnaph.html
