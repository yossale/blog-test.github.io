---
title: Default Jackson serde in Spring
---
## Problem
We're using Spring in most of our Java applications, and rely on Swagger for a lot of API calls. Now, having all the jackson annotations makes everything easy - until it becomes annoying, like this:

```java
    @JsonFormat(pattern = ModelConst.JSON_DATE_FORMAT)
    private LocalDate startDate;
    @JsonFormat(pattern = ModelConst.JSON_DATE_FORMAT)
    private LocalDate endDate;
    @JsonFormat(pattern = ModelConst.JSON_DATE_FORMAT)
    private LocalDate paymentDate; 
    @JsonFormat(pattern = ModelConst.JSON_DATE_FORMAT)
    private LocalDate registrationDate; 
```

Now, having one of these is nice and tidy. Having them all over the model code is annoying, to say the least. 

## So, what do you do? 
Well, with Jackson you can add custom serializers to the ObjectMapper object. Problem is, we're using spring, so we don't really have access to the ObjectMapper object used to do the serialization/deserialization. 

The first option was to generate a bean to generate the ObjectMapper , and assign the serializers to it. However, there's a better way - We can use a configuration bean supplied by Spring:

```java
@Configuration
public class AppConfig {
 
    private static final String dateFormat = "yyyy-MM-dd";
    private static final String dateTimeFormat = "yyyy-MM-dd HH:mm:ss";
 
    @Bean
    public Jackson2ObjectMapperBuilderCustomizer jsonCustomizer() {
        return builder -> {
            builder.serializers(new LocalDateTimeSerializer(
                DateTimeFormatter.ofPattern(DATE_TIME_FORMAT)));
            builder.serializers(new LocalDateSerializer(
                DateTimeFormatter.ofPattern(DATE_FORMAT)));
        };
    }

}
```

This way, we don't need any of the annotations, since we've set the default parsing. 

```java
    private LocalDate startDate;
    private LocalDate endDate;
    private LocalDate paymentDate; 
    private LocalDate registrationDate; 
```