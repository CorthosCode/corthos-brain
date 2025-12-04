
HATEOAS расшифровывается как Hypermedia as the Engine of Application State (Гипермедиа как средство изменения состояния приложения).

Spring HATEOAS – это небольшой проект, который позволяет создавать API, следующие принципу HATEOAS отображения связанных ссылок для данного ресурса.

Принцип HATEOAS гласит, что API должен помогать клиенту, возвращая с каждым ответом службы информацию о возможных следующих шагах.

Это не является основной или обязательной функцией, но если вы хотите организовать для своих клиентов исчерпывающую помощь по всем службам API для данного ресурса, то это отличный вариант.

С помощью Spring HATEOAS можно быстро создавать классы моделей со ссылками на ресурсы, предоставляемые этими моделями.

В Spring HATEOAS также имеется построитель для создания конкретных ссылок, указывающих на методы контроллера Spring MVC.

В следующем фрагменте кода показан пример, как HATEOAS должен представлять службу лицензий:

```json
"_links": {
   "self" : {
    "href" : "http://localhost:8080/v1/organization/optimaGrowth/license/0235431845"
   },
   "createLicense" : {
    "href" : "http://localhost:8080/v1/organization/optimaGrowth/license"
   },
   "updateLicense" : {
    "href" : "http://localhost:8080/v1/organization/optimaGrowth/license"
   },
   "deleteLicense" : {
    "href" : "http://localhost:8080/v1/organization/optimaGrowth/license/0235431845"
   }
}
```


Модель данных:

```java
@Getter @Setter @ToString
public class License extends RepresentationModel < License > {
  private int id;
  private String licenseId;
  private String description;
  private String organizationId;
  private String productName;
  private String licenseType;
}
```


В java классе контроллера настройка выглядит так:

```java
@RequestMapping(value = "/{licenseId}", method = RequestMethod.GET)
public ResponseEntity < License > getLicense(
  @PathVariable("organizationId") String organizationId,
  @PathVariable("licenseId") String licenseId) {
  
  License license = licenseService.getLicense(licenseId,
    organizationId);
    
  license.add(linkTo(methodOn(LicenseController.class)
      .getLicense(organizationId, license.getLicenseId()))
    .withSelfRel(),
    linkTo(methodOn(LicenseController.class)
      .createLicense(organizationId, license, null))
    .withRel("createLicense"),
    linkTo(methodOn(LicenseController.class)
      .updateLicense(organizationId, license))
    .withRel("updateLicense"),
    linkTo(methodOn(LicenseController.class)
      .deleteLicense(organizationId, license.getLicenseId()))
    .withRel("deleteLicense"));
    
  return ResponseEntity.ok(license);
}
```

Метод `add()` – это метод класса `RepresentationModel`.

Метод `linkTo` проверяет класс `LicenseController` и получает корневое представление, а метод `methodOn` получает представление метода, выполняя фиктивный вызов целевого метода.

Оба метода являются статическими методами `org.springframework.hateoas.server.mvc.WebMvcLinkBuilder.WebMvcLinkBuilder` – это вспомогательный класс, предназначенный для создания ссылок на классы контроллеров.

