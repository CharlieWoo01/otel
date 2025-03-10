# otel

Extend autoconfiguration within microservices

```xml
<dependencies>
  <!-- Spring Boot Auto-Configuration -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-autoconfigure</artifactId>
  </dependency>

  <!-- Spring Boot Configuration Processor for metadata generation -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
  </dependency>

  <!-- OpenTelemetry SDK -->
  <dependency>
    <groupId>io.opentelemetry</groupId>
    <artifactId>opentelemetry-sdk</artifactId>
    <version>1.24.0</version> <!-- Use the latest stable version -->
  </dependency>

  <!-- OpenTelemetry OTLP HTTP Exporter (using protobuf) -->
  <dependency>
    <groupId>io.opentelemetry</groupId>
    <artifactId>opentelemetry-exporter-otlp-http</artifactId>
    <version>1.24.0</version>
  </dependency>
</dependencies>


```


```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.otel.autoconfigure.OpenTelemetryAutoConfiguration
```

```java
package com.example.otel.config;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties(prefix = "otel")
public class OTELProperties {

    /**
     * The OTLP HTTP endpoint URL.
     * For example: http://localhost:4318/v1/traces
     */
    private String endpoint = "http://localhost:4318/v1/traces";

    // Additional properties (timeouts, headers, etc.) can be added here

    public String getEndpoint() {
        return endpoint;
    }

    public void setEndpoint(String endpoint) {
        this.endpoint = endpoint;
    }
}
```

```
package com.example.otel.autoconfigure;

import com.example.otel.config.OTELProperties;
import io.opentelemetry.api.OpenTelemetry;
import io.opentelemetry.exporter.otlp.http.trace.OtlpHttpSpanExporter;
import io.opentelemetry.sdk.OpenTelemetrySdk;
import io.opentelemetry.sdk.trace.SdkTracerProvider;
import io.opentelemetry.sdk.trace.export.SimpleSpanProcessor;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableConfigurationProperties(OTELProperties.class)
@ConditionalOnClass({OpenTelemetrySdk.class, OtlpHttpSpanExporter.class})
public class OpenTelemetryAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public OpenTelemetry openTelemetry(OTELProperties properties) {
        // Build the OTLP HTTP exporter using the provided endpoint.
        OtlpHttpSpanExporter exporter = OtlpHttpSpanExporter.builder()
                .setEndpoint(properties.getEndpoint())
                .build();

        // Create a tracer provider with a simple span processor.
        SdkTracerProvider tracerProvider = SdkTracerProvider.builder()
                .addSpanProcessor(SimpleSpanProcessor.create(exporter))
                .build();

        // Build and register the global OpenTelemetry instance.
        return OpenTelemetrySdk.builder()
                .setTracerProvider(tracerProvider)
                .buildAndRegisterGlobal();
    }
}
```