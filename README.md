# Simple Query Handler

A simplified demonstration of a production-level query generation system using the Strategy Pattern. This project shows how to build SQL queries dynamically from HTTP requests using a clean, extensible architecture.

## 🔄 Request-to-SQL Flow

The system transforms HTTP requests into SQL queries through this clear pipeline:

```
HTTP Request → FastAPI Route → QueryParameters → QueryBuilder → SQL Response
     ↓              ↓               ↓              ↓           ↓
 JSON Body    →  Validation   →  Data Model  →  SQL Logic  → JSON with SQL
```

**Detailed Flow:**
1. **HTTP Request**: Client sends POST request with filter parameters
2. **FastAPI Route**: `/simple-query` endpoint receives and validates request
3. **QueryParameters**: Pydantic model structures the request data
4. **QueryBuilder**: Core component that orchestrates SQL generation
5. **SQL Response**: Generated SQL returned in JSON format

## 🏗️ Architecture Deep Dive

The QueryBuilder follows this internal process:
```
QueryBuilder.build_query(request_data)
    ↓
Creates QueryDefinition
    ↓
QueryDefinition.handle_parameters(request_data)
    ↓
Strategies process each parameter type
    ↓
SQL components collected (WHERE, columns, etc.)
    ↓
Template substitution creates final SQL
    ↓
SQL returned to client
```

## 🚀 Quick Start

1. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

2. **Start the server:**
   ```bash
   python main.py
   ```

3. **Send HTTP request to generate SQL:**
   ```bash
   curl -X POST "http://localhost:8080/simple-query" \
     -H "Content-Type: application/json" \
     -d '{
       "table": "products",
       "filters": {
         "status": {"values": ["active"], "operator": "IN"}
       }
     }'
   ```

4. **Receive SQL response:**
   ```json
   {
     "query_id": "uuid-123",
     "sql": "SELECT * FROM products WHERE status IN ('active')",
     "message": "Query generated successfully"
   }
   ```

## 📝 Request → SQL Examples

### Example 1: Basic Filter Request
**HTTP Request:**
```json
POST /simple-query
{
  "table": "users",
  "filters": {
    "status": {"values": ["active", "pending"], "operator": "IN"}
  }
}
```
**Generated SQL:**
```sql
SELECT * FROM users WHERE status IN ('active', 'pending')
```

### Example 2: Complex Request with Multiple Parameters
**HTTP Request:**
```json
POST /simple-query
{
  "table": "products",
  "columns": ["id", "name", "price"],
  "filters": {
    "category": {"values": ["electronics"], "operator": "="},
    "price": {"values": ["100"], "operator": ">="}
  },
  "order_by": ["name", "price DESC"],
  "limit": 50
}
```
**Generated SQL:**
```sql
SELECT id, name, price 
FROM products 
WHERE category = 'electronics' AND price >= '100' 
ORDER BY name, price DESC 
LIMIT 50
```

## 🔧 QueryBuilder: The Heart of SQL Generation

The `QueryBuilder` is the central component that:
1. **Receives** validated request data from FastAPI routes
2. **Creates** appropriate QueryDefinition instance
3. **Orchestrates** the strategy-based SQL building process
4. **Returns** formatted response with generated SQL

```python
# This is what happens when you send an HTTP request:
def build_query(self, query_parameters: QueryParameters) -> SimpleQueryResponse:
    # 1. Create query definition for this request type
    query_definition = SimpleQueryDefinition()
    
    # 2. Process request parameters through strategies
    sql = query_definition.build_query(query_parameters)
    
    # 3. Generate tracking ID and format response
    query_id = str(uuid.uuid4())
    
    # 4. Return SQL to client
    return SimpleQueryResponse(query_id=query_id, sql=sql, message="success")
```

## 📚 Architecture Benefits

1. **Extensible**: Add new operators by creating new strategies
2. **Maintainable**: Clear separation of concerns
3. **Testable**: Each component can be tested independently
4. **Production-Ready Pattern**: Based on real production architecture

## 🗂️ Project Structure

```
app/
├── models/                    # Data models
│   ├── query_parameters.py   # Input model
│   └── simple_models.py      # Response model
├── query_generation/          # Core query building logic
│   ├── strategies/           # Strategy pattern implementations
│   ├── query_definitions/    # Query orchestration
│   ├── query_builders/       # Final assembly
│   └── templates/           # SQL templates
└── routes/                   # API endpoints
    └── simple_query.py      # REST endpoints
```

## 🎯 Design Patterns

- **Strategy Pattern**: Different SQL generation strategies
- **Template Method**: Query building process
- **Abstract Factory**: Query builder creation

## 📖 API Documentation

Once the server is running, visit:
- **Swagger UI**: http://localhost:8080/docs
- **ReDoc**: http://localhost:8080/redoc

## 🧪 Testing

Use the provided HTTP file:
```bash
# Test with your HTTP client
api_requests/simple_query.http
```

---