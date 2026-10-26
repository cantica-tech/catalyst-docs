# How the catalyst projects fit together

```mermaid
graph LR
    subgraph Core
        K["catalyst<br/>(kernel + CLI)"]
    end

    subgraph Modules["Process modules"]
        SE["catalyst-software-engineering<br/>(SE module)"]
        KBM["catalyst-knowledge-base<br/>(KBM module)"]
    end

    subgraph Tools["Products and services"]
        UI["catalyst-ui<br/>(VS Code + Electron Studio)"]
        ENG["catalyst-kb-engine<br/>(KB onboarding and builds)"]
        USR["catalyst-users<br/>(IAM facade, Keycloak)"]
    end

    subgraph KBs["Knowledge base"]
        DOCS["catalyst-docs<br/>(public docs site)"]
    end

    DIST["cantica-tech<br/>(release distribution)"]

    SE -- plugs into --> K
    KBM -- plugs into --> K

    UI -- reads criteria, drives --> K
    USR -- signs deployments --> K
    UI -- sign-in --> USR
    ENG -- executes workflows of --> KBM

    DOCS -- built with --> KBM
    DOCS -- built by --> ENG
    DOCS -- documents --> K

    K -- released to --> DIST
    SE -- released to --> DIST
    KBM -- released to --> DIST

    K -. governs, SE module .-> UI
    K -. governs, SE module .-> ENG
    K -. governs, SE module .-> USR
    K -. governs, SE module .-> KBM
```

Solid arrows: runtime or build dependencies. Dotted arrows from `catalyst`:
the project is itself a catalyst deployment (its criterion uses the SE
module); `catalyst-docs` is a deployment using KBM alone.
