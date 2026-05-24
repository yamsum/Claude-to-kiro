## Jenkins Groovy Review

When reviewing Jenkinsfiles or shared library Groovy (`.groovy` files in `vars/` or `src/`):

**CPS violations (runtime failures):** Methods using Java streams (`.stream()`, `.filter()`, `.map()`, `.collect()`), closures passed to non-pipeline methods, or `Iterator` must be annotated `@NonCPS`. Flag every occurrence.

**Credential exposure (CRITICAL):** Any `"${env.VARIABLE_NAME}"` or `"${params.ANYTHING}"` inside `sh()`, `bat()`, or string concatenation is a secret leak. Credentials must use `withCredentials([...])` binding with **single-quoted** strings in the `sh()` call.

**Sandbox blocks:** Flag calls to `System.*`, `Runtime.exec()`, `new File(...)`, reflection (`getClass()`, `getDeclaredMethod()`), `Thread`, or any third-party import not on the approved classpath. These cause `RejectedAccessException` at runtime in untrusted pipelines.

**Serialization:** Top-level pipeline variables must be serializable. Flag `new groovy.json.JsonSlurper()` (use `JsonSlurperClassic`), Groovy closures stored at pipeline scope, and custom classes without `implements Serializable`.

**Declarative structure:** Every pipeline must have `timeout()` in `options {}`. No `node {}` blocks inside `pipeline {}`. `parallel {}` stages must not write to shared paths.

**Shared library blast radius:** For changes to `vars/*.groovy` or `src/**/*.groovy`, report: how many pipelines call this step, whether the change is backward-compatible, and whether the library is tag-pinned or branch-pinned in consumers.

Format findings: `[CRITICAL|IMPORTANT|NIT] Category — file.groovy ~line N: problem. Fix: action.`
