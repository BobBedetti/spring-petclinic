# Testing Spring PetClinic

## Prerequisites

- Java 21 or newer (full JDK)
- Maven wrapper included in repo (`./mvnw`)

## Devin Secrets Needed

None — the app uses an embedded H2 in-memory database with no authentication required.

## Quick Start

```bash
# Build and verify (runs tests, checkstyle, spring-javaformat validation)
./mvnw verify

# Run the application
./mvnw spring-boot:run
# App available at http://localhost:8080
```

## Key Endpoints

| Endpoint | Description |
|---|---|
| `http://localhost:8080/` | Home page |
| `http://localhost:8080/vets.html` | Vets list (HTML, paginated, 5 per page) |
| `http://localhost:8080/vets` | Vets API (XML by default, JSON with `Accept: application/json`) |
| `http://localhost:8080/owners/find` | Owner search form |
| `http://localhost:8080/owners/{id}` | Owner detail with pets and visits |
| `http://localhost:8080/actuator/info` | Build info including Java source/target version |
| `http://localhost:8080/actuator/env/{property}` | Environment properties (values may be masked with `******`) |

## Testing Strategy

### Vet Specialties (Critical Path)
The vet specialties sort order is the most important thing to verify when changing collection/stream code:
- **Linda Douglas** (id=3) has two specialties: surgery (id=2) and dentistry (id=3)
- They should appear alphabetically: "dentistry surgery" on the HTML page, and `["dentistry", "surgery"]` in the JSON API
- If they appear as "surgery dentistry", the sort in `Vet.getSpecialties()` is broken

### JSON API Contract
```bash
curl -s -H "Accept: application/json" http://localhost:8080/vets | python3 -m json.tool
```
Expected schema: `{ "vetList": [{ "id", "firstName", "lastName", "new", "nrOfSpecialties", "specialties": [{ "id", "name", "new" }] }] }`
Expect 6 vets total (James Carter, Helen Leary, Linda Douglas, Rafael Ortega, Henry Stevens, Sharon Jenkins).

### Owner Search Regression
- Search for "Davis" → expect Betty Davis (pet: Basil, hamster) and Harold Davis (pet: Iggy)
- Click into Betty Davis → verify detail page shows address, city, telephone, pet info

### Java Version Verification
Use actuator info endpoint rather than `/actuator/env/java.runtime.version` (values are masked):
```bash
curl -s http://localhost:8080/actuator/info | python3 -m json.tool
```
Look for `"java": { "source": "21", "target": "21" }` in the `build` section.

## Common Issues

- **Port 8080 already in use**: Kill existing process with `fuser -k 8080/tcp` before starting
- **Spring Java Format violations**: Run `./mvnw spring-javaformat:apply` to auto-fix, then commit the changes
- **Actuator env values masked**: Use `/actuator/info` instead of `/actuator/env/{property}` for build/Java version info
- **Checkstyle**: Currently pinned to v12.x; upgrading to v13+ may introduce unrelated style changes
- **H2 database**: Data is seeded at startup and lost on restart — no persistent state to worry about
