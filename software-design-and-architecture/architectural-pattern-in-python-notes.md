# MVC vs DDD vs 3-tier Architecture

## MVC

Short for Model-View-Controller, a very popular architecture pattern that seperate the software system into 3 components.

However, due to the complexity of modern-day software system, a hybrid variant of MVC introduces a layer called "Service". By original design, it was the Model that represents the data and the business logic, but "Model" is more aligned with the "data" and "data-related logic" more that "business logic", therefore "Service" is born.

### Service

- Sits between Model and Controller.
- Handle most of the business logic involves working with multiple Models and external systems communication.

  - Flow: Controller calls the Services to return the Models.

- Model:

  - Handle data structure, validation, and database interaction via ORM or direct (i.e. SQL)
  - Sometimes handle business logic, but only the logic that stays within the context/boundary of that Model.
  - One Model can include other Models.
  - Flow: The Models are returned by the Services.
  - NOTE: Model in MVC is different from Model in DDD which are Domain Objects.

### Controllers

- A.k.a the orchestrator.
- IT SHOULD NOT CONTAIN BUSINESS LOGIC, IT SHOULD ONLY BE ABLE TO CALL TO SERVICES. Services is the one to store business logic.
- Controllers' complexity is heavily dependent on how engineers fetch Model to build View.
- Flow:

1. Receives data from users.
2. Calls to Services with data as input.
3. Services return Models as output.
4. Using Models, Controllers build Views.
5. Returns Views to users.

- Structure:

```
.tmp/             # code that will not be version-controlled
shared/           # shared code between repositories (via Git subtree/Git submodule)
instance/         # .env files, mysql.cnf, redis.conf
src/
├── apis/         # or `controllers/`, `routes/`, `endpoints/`. You can use single module `routes.py`, `endpoints.py` if smaller scope
│   ├── v1/       # version 1 APIs, store blueprints
│   ├── v2/       # version 2 APIs, store blueprints
│   └── ...
├── configs/      # or `config.py` if smaller scope, store non-sensitive configurations
├── constants/    # or `data/`, store static data
├── models/       # store data structure, validation, ORM/direct and some minimal business logic
├── services/     # most of business logic, take Model as dependency
├── static/       # CSS, JS, images, videos, PDFs, ...
├── templates/    # store HTML page that required SEO.
├── utils/        # or `middleware/`, store reusable code (pure functions, decorators, loggers, validators, error handlers, authenticators, ...)
├── (end of directory) # ================================================
├── extensions.py # Flask/Quart extensions initialization
├── exception_handler.py
├── error_handler.py
└── ...
```

- Tips: by seperating `apis` and `views` into 2 seperate folders, documenting process via Swagger is also separated and can be fully automated without needing to write a single line of code.

### Models

-

### Views

- Presentation Layer, User Interface, data visible to the system...
- Analogy: In the context of Flask framework, Views are JSON data built by `jsonify()` or Jinja2 HTML file built by `make_template()`, ...

## DDD

## 3-tier architecture

Presentation Tier
|
v
Business Logic Tier (Service, Entities, Data Access Layer, ...)
|
v
Data Tier

## References

- [youngmonkeys/ezyplatform-development EzyPlatform Admin SDK](https://github.com/youngmonkeys/ezyplatform-development/tree/master/ezyplatform-sdk/ezyplatform-admin-sdk/src/main/java/org/youngmonkeys/ezyplatform/admin)

- [youngmonkeys/ezyplatform-development EzyPlatform Web SDK](https://github.com/youngmonkeys/ezyplatform-development/tree/master/ezyplatform-sdk/ezyplatform-web-sdk/src/main/java/org/youngmonkeys/ezyplatform/web)
