---
name: Junit Platform Alignment
---

# Alinhamento de Versões JUnit Platform

## Problema
`cucumber-junit-platform-engine:7.20.1` puxa `junit-platform-engine:1.11.2`, mas `junit-platform-suite:1.11.4` espera `1.11.4`

## Solução
Excluir transitive e declarar explicitamente:

```xml
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-junit-platform-engine</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.junit.platform</groupId>
            <artifactId>junit-platform-engine</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.junit.platform</groupId>
    <artifactId>junit-platform-engine</artifactId>
    <version>${junit.platform.version}</version>
</dependency>
```
