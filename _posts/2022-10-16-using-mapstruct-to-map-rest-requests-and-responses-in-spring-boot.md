---
layout: post
title: Using MapStruct To Map REST Requests And Responses In Spring Boot
author: Carlos Molero Mata
description: In this post we are going to learn how to use mapstruct library to map requests and responses in Spring Boot applications.
header-style: text
tags: [java, springboot]
---

Today we are going to learn how to map our `JPA` Entities to our DTO classes, this technique is often referred as **The DTO Pattern**. The pattern was first introduced by Martin Fowler in his book [EAA](https://martinfowler.com/books/eaa.html).

DTO's are flat data structures that contains no business logic, they only contain storage, accessors and eventually methods related to serialization or parsing.

This technique is useful when:

- Your system has a lot of remote calls
- Your domain model is composed of many different objects
- You need to build different views from your domain models

## What is MapStruct?

From the docs:

> MapStruct is a code generator that greatly simplifies the implementation of mappings between Java bean types based on a convention over configuration approach.

Multi-layered applications often require to map between different object models (e.g. entities and DTOs). Writing such mapping code is a tedious and error-prone task. MapStruct aims at simplifying this work by automating it as much as possible.

In contrast to other mapping frameworks MapStruct generates bean mappings at compile-time which ensures a high performance, allows for fast developer feedback and thorough error checking.

## Example

### Handling requests

Let's start with this JPA Entity model:

```java
@Builder
@Data
@Entity
@NoArgsConstructor
@AllArgsConstructor
@Table(name = "products")
public class ProductEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    private String name;
    private String description;
    private Float price;
    private String imgUrl;
    @CreationTimestamp
    private Instant createdAt;
    @UpdateTimestamp
    private Instant updatedAt;
}
```

This is our entity, an entity is a model that belongs to the persistence layer and has specific JPA annotations which mark special fields like id, createdAt and updatedAt.

Imagine that we have an endpoint in our REST API that handles the creation of a product. In this case, we only want to request name, description, price and imgUrl so let's create a DTO class with only those fields.

```java
@Data
public class ProductCreateRequest {
    @NotNull(message = "name should not be null")
    @NotEmpty(message = "name should not be empty")
    private String name;
    @NotNull(message = "description should not be null")
    @NotEmpty(message = "description should not be empty")
    private String description;
    @NotNull(message = "price should not be null")
    private Float price;
    @NotNull(message = "imgUrl should not be null")
    @NotEmpty(message = "imgUrl should not be empty")
    private String imgUrl;
}
```

This class doesn't have any business logic, just getters and setters. It obeys a single purpose, to be the transfer object which holds and validates the information sent by the client.

After receiving the request in our endpoint, we need to map it to our entity class to let the persistence layer store the product.

The communication with the persistence layer is achieved with the help of a service which implements a predefined interface. Our service interface declares a method to create a new product, here is where the service or business layer communicates with the persistence layer.

```java
@Service
public class ProductServiceImpl implements ProductService {
    @Override
    public ProductEntity createProduct(ProductEntity product){
        return productRepository.save(product);
    }
}
```

Now we are going to create a mapper called `ProductRestMapper`. We import the `@Mapper` annotation from `org.mapstruct.Mapper;` and declare a method which receives an instance of `ProductCreateRequest` and returns an instance of `ProductEntity`. We don't need to do anything else, from here everything will be handled by `MapStruct`!

```java
@Mapper
public interface ProductRestMapper {
    ProductEntity fromProductCreateRequestToProductEntity(ProductCreateRequest product);
}
```

When we receive a request in our endpoint we can map it to a product entity that we can pass to the `createProduct()` method of our service that will communicate with our persistence layer and store it.

Do you want to know how to respond to the client? Read on!

### Handling responses

Ok, now imagine for a second that we aren't talking about a product entity, imagine we are talking about an user entity.

The `UserEntity` class would probably hold some sensitive information like a password that we don't want to expose to the client in the response. How would we avoid this? Or how do we return additional information such as a jwt token when the user signs in?

For this purpose, we could, of course, use the `@JsonIgnore` annotation in the password field and add an `authToken` field (which will remain null in our database) to our entity to bypass this problem. However, this is not convenient since we are polluting our entity, mixing application layers and responsabilities and losing flexibility.

You're guessing right, the DTO pattern can help us with this problem, **we will need to create an additional DTO model to handle the response**.

```java
@Data
public class ProductCreateResponse {
    private Long id;
    private String name;
    private String description;
    private Float price;
    private String imgUrl;
}
```

This class is slightly different from the `ProductCreateRequest` one, we added the id because the client will probably need to pass the id in other requests such as update or delete requests. Also, we are not including the createdAt and the updatedAt fields as we don't need them for this particular case. This DTO will allow us to control what data is returned to the client from our endpoint.

As you are probably guessing, we need to add a new method declaration to our rest mapper:

```java
ProductCreateResponse fromProductEntityToProductCreateResponse(ProductEntity product);
```

Finally, our endpoint will look like this:

```java
@PostMapping("/create")
public ResponseEntity<ProductCreateResponse> createProduct(@RequestBody ProductCreateRequest productCreateRequest){
    ProductEntity product = productRestMapper.fromProductCreateRequestToProductEntity(productCreateRequest);
    ProductEntity createdProduct = productService.createProduct(product);
    ProductCreateResponse productCreateResponse = productRestMapper.fromProductEntityToProductCreateResponse(createdProduct);
    return ResponseEntity.ok(productCreateResponse);
}

```

## Conclusion

With this pattern we can have different 'views' and 'inputs' for our domain model and we achieve the creation of a predictable and self-validating communication mechanism between the client and the server, proper code and layer splitting and separation of responsabilities.

Btw, if you are using `Lombok` with `MapStruct` you probably want to check this library: [lombok-mapstruct-binding](https://mvnrepository.com/artifact/org.projectlombok/lombok-mapstruct-binding).
