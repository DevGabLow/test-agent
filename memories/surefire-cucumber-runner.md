---
name: Surefire Cucumber Runner
---

# Surefire + Cucumber Runner

## Naming: Runner DEVE bater com padrão Surefire
- `CucumberTestSuite.java` **NÃO** é descoberto
- Renomear para `RunCucumberTest.java` (bate com `*Test.java`)

## @SelectClasspathResource NÃO funciona com Cucumber
- `@SelectClasspathResource("features")` espera arquivo de texto, não diretório
- Usar: `@ConfigurationParameter(key = FEATURES_PROPERTY_NAME, value = "classpath:features")`
