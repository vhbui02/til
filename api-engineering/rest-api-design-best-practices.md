# REST API Design Best Practices

One of the crucial design decisions during SDLC is REST API design with a strong and consistent REST resource naming strategy.

## Overview

### Resource

A **Resource** is an object that's important enough to be referenced in itself. It has:

- data
- relationships to other resources.
- methods operating against it to allow for access/manipulate info

A group of Resources is called a **Collection**. A Collection can be nested inside another Collection, those are called **Sub-Collection**. However, when a resource is the smallest data unit, it's called a **Singleton**.

```
/customers                                          # collection resource
/customers/{{customer-id}}/accounts                 # sub-collection resource
/customers/{{customer-id}}/accounts/{{account-id}}  # singleton resource
```

The URI is the online address of a Resource. REST API designers should create URIs that convey a **REST API resource model** to clients of the API.

**THREE archetypes of a Resource:**

1. **Document:** object instance, database record, ... They should be named in **singular** form. `GET` filter will be chosen from their obj property/database column.

```
/about-us                       # document
/contact-us                     # document
/users/{id}/status              # document
/users/{id}/profile             # document
```

2. **Collection:** group of resources _managed by the server_. When a client wants to add a new resource, it sends a request to the collection, and the server decides whether to create the resource and assign it a new URI.

Collections come in two types:

- **Primary:** The main group of resources (e.g., `/users`, `/customers`).
- **Secondary:** A sub-group related to a specific resource in the primary collection (e.g., `/users/{id}/posts`, `/customers/{id}/accounts`).

```
/users                            # primary collection
/customers                        # primary collection
/users/{id}/posts                 # secondary collection
/customers/{id}/accounts          # secondary collection
```

3. **Store (less popular):** client-managed directory. Store resource put resources in, get them back out, decide when to delete them. A store **never** generate new URIs, instead each stored resource has a URI and this URI is chosen by the client.

```
/users/{id}/playlists             # store
```

### Requests

- Handle the resource relationships with **parameters:** query parameters, path parameters.

### Responses

1. Client + API success - `2xx` code.
2. Client error - `4xx` code.
3. API error - `5xx` code.

Describe error responses concise and neat, enough info for end user to work on fixing the cause but not too much to avoid leaking sensitive information.

### Benefits

- High readability and comprehensibility, memorable.
- Less likely to write incorrect code.
- Scalable/Extendable.
- Add to application's completeness.

### Experiences

- Think about your use cases carefully before designing. What is the insight/analysis that can be derived from data that you want to show to your users?
- API endpoints ~ Features.
- API design is an iteration process.

- Use forward slash `/` to indicate hierarchical relationships. DO NOT use trailing forward slash `/`:

  ```
  /users/                           # bad!

  /users                            # good
  /users/{id}                       # good
  /users/{id}/comments              # good
  ```

- Write in `kebab-case` to improve readability. DO NOT write in `camelCase` or `snake_case`.

  ```
  /devicemanagement/manageddevices       # bad!
  /device_management/managed_devices     # bad!
  /deviceManagement/managedDevices       # bad!

  /device-management/managed-devices     # good
  ```

- Use ALL lowercase letters. DO NOT use `UPPERCASE`. `Capitalize` or `camelCase`.

  ```
  /My-Folder/My-Doc               # bad
  /MY-FOLDER/MY-DOC               # bad
  /myFolder/myDoc                 # bad

  /my-folder/my-doc               # good
  ```

- Use query components to filter Collection resource. DO NOT create new URI.

  ```
  /device-management/managed-devices/usa-region     # bad
  /device-management/managed-devices?region=USA     # good
  ```

- DO NOT use file extensions

  ```
  /device-management/managed-devices.xml            # bad
  /device-management/managed-devices                # good
  ```

- Use a noun with an implicit associated action. For CRUD operation, **DO NOT use verbs in the URI**. Remember: Resource document archtype is perfectly fine.

  ```
  /device-management/managed-devices/{id}/scripts/{id}/execute      # bad
  /device-management/managed-devices/{id}/scripts/{id}/status       # good
  ```

  > **NOTE:** Google Cloud introduces an API design that use colon `:` which is called custom method.

- REST API using OO design concept for more granularity is perfectly fine. It fasilitate the use of object's method, which allows for more operatios beside CRUD.

  ```
  /domain/resource/operation/qualifier/parameter
  ```

## FOUR step process of REST API Designing with examples

Goal: Design API endpoints for a network-based application.

### Step 1: Object Modeling

Identify the object that will be presented as resources.
Identify the unique identifier for each object.

```
devices
routers
modems
switches
configurations
...etc.
```

- TWO categories: _devices_ and _configurations_.
- ONE device can have MANY configuration.
- Device is a top-level resource, configuration is a sub-resource under the device.
- Configuration can also be a top-level resource.
- Use `id` whose data type is integer/UUID/NanoID/... as unique identifier.

### Step 2: Create Model URIs

Focus on the relationship between resources and their sub-resources.

> A resource URI can be called an **API endpoint**.

```
/devices
/devices/{id}

/configurations
/configurations/{id}

/devices/{id}/configurations
/devices/{id}/configurations/{id}
```

### Step 3: Determine resource represenations

What's the MIME type of your HTTP Response Body?

**XML** used to be in the game but not anymore. Nowadays, most representations are defined in **JSON**.

#### 3.1. `/devices` - a Collection Resource listing the basic information of multiple devices

```json
{
  "devices": {
    "size": "2",
    "link": {
      "rel": "self",
      "href": "/devices"
    },
    "device": [
      {
        "id": "12345",
        "link": {
          "rel": "self",
          "href": "/devices/12345"
        },
        "deviceFamily": "apple-es",
        "OSVersion": "10.3R2.11",
        "platform": "SRX100B",
        "serialNumber": "32423457",
        "connectionStatus": "up",
        "ipAddr": "192.168.21.9",
        "name": "apple-srx_200",
        "status": "active"
      },
      {
        "id": "556677",
        "link": {
          "rel": "self",
          "href": "/devices/556677"
        },
        "deviceFamily": "apple-es",
        "OSVersion": "10.3R2.11",
        "platform": "SRX100B",
        "serialNumber": "6453534",
        "connectionStatus": "up",
        "ipAddr": "192.168.20.23",
        "name": "apple-srx_200",
        "status": "active"
      }
    ]
  }
}
```

Collection resource only contains the most important information about its Singular resources.

This will keep the size of the HTTP Response Body (payload) small, therefore improve the performance of the API.

#### 3.2. `/devices/{id}`: a Singular Resource specifying the detailed information of an individual device

```json
{
  "id": "12345",
  "link": {
    "rel": "self",
    "href": "/devices/12345"
  },
  "deviceFamily": "apple-es",
  "OSVersion": "10.0R2.10",
  "platform": "SRX100-LM",
  "serialNumber": "32423457",
  "name": "apple-srx_100_lehar",
  "hostName": "apple-srx_100_lehar",
  "ipAddr": "192.168.21.9",
  "status": "active",
  "configurations": {
    "size": "2",
    "link": {
      "rel": "self",
      "href": "/configurations"
    },
    "configuration": [
      {
        "id": "42342",
        "link": {
          "rel": "self",
          "href": "/configurations/42342"
        }
      },
      {
        "id": "675675",
        "link": {
          "rel": "self",
          "href": "/configurations/675675"
        }
      }
    ]
  },
  "method": [
    {
      "href": "/devices/12345/exec-rpc",
      "rel": "rpc"
    },
    {
      "href": "/devices/12345/synch-config",
      "rel": "synch device configuration"
    }
  ]
}
```

The design of each resource, be that a Singular or a Collection, contains at least one link (e.g. to itself `"href": "/configurations"`) is called **HATEOAS-driven**. A primary collection can have extra links to its secondary collection (configurations, methods).

> A REST API that return a HTML file with anchor tags is also called **HATEOAS-driven**.

#### 3.3. `/configurations`: a Collection resouce listing the basic information of multiple configurations

```json
{
  "configurations": {
    "size": "20",
    "link": {
      "rel": "self",
      "href": "/configurations"
    },
    "configuration": [
      {
        "id": "42342",
        "link": {
          "rel": "self",
          "href": "/configurations/42342"
        }
      },
      {
        "id": "675675",
        "link": {
          "rel": "self",
          "href": "/configurations/675675"
        }
      }
      // ... more configurations would follow
    ]
  }
}
```

As you can see, there are 20 configurations for various devices. But there are only 2 configurations for each device.

#### 3.4. `/configurations/{id}`: a Singular resource specifying the detailed information of an individual configuration

```json
{
  "id": "42342",
  "link": {
    "rel": "self",
    "href": "/configurations/42342"
  },
  "content": "…",
  "status": "active",
  "links": [
    {
      "rel": "very big raw configuration script",
      "href": "/configurations/42342/raw"
    }
  ]
}
```

It includes all possible information about a Configuration, including a link relevant links. It is also called the resource from the _primary collection_.

#### 3.5. `/devices/{id}/configurations`: a Collection resource of Configuration under a Singular collection of Device

```json
{
  "configurations": {
    "size": "2",
    "link": {
      "rel": "self",
      "href": "/devices/12345/configurations"
    },
    "configuration": [
      {
        "id": "53324",
        "links": [
          {
            "rel": "self",
            "href": "/devices/12345/configurations/53324"
          },
          {
            "rel": "detail",
            "href": "/configurations/53324"
          }
        ]
      },
      {
        "id": "333443",
        "links": [
          {
            "rel": "self",
            "href": "/devices/12345/configurations/333443"
          },
          {
            "rel": "detail",
            "href": "/configurations/333443"
          }
        ]
      }
    ]
  }
}
```

Representations can have extra links:

- one acts as its direct representation inside sub-collection: `/devices/12345/configurations/53324`
- one acts as its location in the primary collection: `/configurations/53324`

#### 3.6. `/devices/{id}/configurations/{id}`: a Singular resource of Configuration under a Singular resource of Device

```json
{
  "id": "11223344",
  "link": {
    "rel": "self",
    "href": "/devices/12345/configurations/11223344"
  },
  "content": "…",
  "status": "active",
  "links": [
    {
      "rel": "detail",
      "href": "/configurations/11223344"
    },
    {
      "rel": "raw configuration content",
      "href": "/configurations/11223344/raw"
    }
  ]
}
```

Either it have the exactly representation as of [the top-level Singular resource of Configuration](design-rest-api-best-practices.md#34-configurationsid-a-singular-resource-of-configuration) or you may mask a few fields,

### Step 4: Assigning HTTP Methods

After deciding all resource URIs and their representation, let's decide the application's possible operation and map those operations to the resource URIs.

E.g. A user can make CRUD operations on devices from the network, on configurations on each device

```
HTTP GET /devices
HTTP GET /configurations

HTTP GET /devices?size=10&page=1        # pagination, filtering
HTTP GET /configurations?size=5?page=2  # pagination, filtering

HTTP GET /devices/{id}
HTTP GET /configurations/{id}

HTTP GET /devices/{id}/configurations
HTTP GET /devices/{id}/configurations/{id}

HTTP POST /devices          # 201 Created
HTTP POST /configurations   # 201 Created

# NOTE:

HTTP PUT /devices/{id}          # 200 OK
HTTP PUT /configurations/{id}
HTTP PUT /devices/{id}/configurations

HTTP DELETE /devices/{id}/configurations/{id}
HTTP DELETE /devices/{id}
HTTP DELETE /configurations/{id}
```

The HTTP POST Request Body payload should not contain `id` attribute, that's for server to decide. A HTTP POST Response should return a `201 Created` status code + message.

The HTTP PUT Response should return a `200 OK` status code + message.

For HTTP DELETE request:

If the deletion operation is synchronous:

- The server completes the deletion before sending the response.
- The resource has been deleted permanently.
- There are 2 appropriate status codes:
  - `200 OK` if the response body includes information about the deleted resource.
  - `204 No Content` if there is no response body.

If the deletion operation is asynchronous:

- The server acknowledge the delete request but places it in a queue for later processing.
- The deletion hasn't actually occured yet when the response is sent.
- The status code `202 Accepted` indicates "I've received your request and will process it later", usually the response body will include a task id that can be tracked for success/failure status.
- The resource may continue to exist for some time after the response.

## References

- [2020-03-02, StackOverflow Blog, "Best practices for REST API design"](https://stackoverflow.blog/2020/03/02/best-practices-for-rest-api-design/)
- [2023-11-04, Lokesh Gupta's "REST API URI Naming Conventions and Best Practices"](https://restfulapi.net/resource-naming/)
- [2023-11-06, Lokesh Gupta's "How to Design a REST API"](https://restfulapi.net/rest-api-design-tutorial-with-example/)
- [Swagger, "Best Practices in API Design"](https://swagger.io/resources/articles/best-practices-in-api-design/)
