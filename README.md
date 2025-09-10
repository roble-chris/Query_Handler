# Simple Query Handler

A simplified demonstration of a production-level query generation system using the Strategy Pattern. This project shows how to build SQL queries dynamically from HTTP requests, following the **exact same architecture** as production systems that execute against Snowflake.

## 🔄 Complete Request-to-SQL Flow (Demo vs Production)

**Your Simplified Version (Demo):**
```
HTTP Request → FastAPI Route → QueryParameters → QueryBuilder → Generated SQL + Demo UUID
     ↓              ↓               ↓              ↓                    ↓
 JSON Body    →  Validation   →  Data Model  →  SQL Logic  →    Demo Response
```

**Production Version:**
```
HTTP Request → FastAPI Route → QueryParameters → QueryBuilder → SQL → Snowflake → Real Query ID
     ↓              ↓               ↓              ↓           ↓        ↓           ↓
 JSON Body    →  Validation   →  Data Model  →  SQL Logic  → Execute → Snowflake → Response
```

## 🎯 **Key Difference: SQL Generation vs Execution**

### **Your Simplified Version (What It Actually Does):**
1. **HTTP Request**: Client sends POST request with filter parameters
2. **FastAPI Route**: `/simple-query` endpoint receives and validates request
3. **QueryParameters**: Pydantic model structures the request data
4. **QueryBuilder**: Generates SQL using the same architecture as production
5. **Demo Response**: Returns generated SQL + UUID (for demonstration only)

### **Production Version (What Production Does):**
1. **HTTP Request**: Backend sends POST request with filter parameters
2. **FastAPI Route**: `/simple-query` endpoint receives and validates request  
3. **QueryParameters**: Pydantic model structures the request data
4. **QueryBuilder**: Generates SQL using strategy pattern
5. **SQL Generation**: Strategies build the complete SQL query
6. **Snowflake Execution**: Generated SQL is executed against Snowflake database
7. **Snowflake Response**: Snowflake returns a unique query_id for tracking
8. **Final Response**: Both the Snowflake query_id and generated SQL returned to client

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
SQL executed against Snowflake database
    ↓
Snowflake query_id + SQL returned to client
```

## 🎯 **Key Point: Snowflake Integration**

**Important**: This simplified version demonstrates the SQL generation architecture but uses dummy Snowflake credentials. In production:

- **SQL is executed against real Snowflake databases**
- **query_id comes from Snowflake's query execution response**  
- **Results are stored in Snowflake and tracked by query_id**
- **Backend can later retrieve results using the Snowflake query_id**

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

## 📝 Request → Snowflake Examples

### Example 1: Basic Filter Request → Snowflake Execution
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
**Snowflake Response:**
```json
{
  "query_id": "b0d47c1d-b195-44a6-a998-3be702b6924e",
  "sql": "SELECT * FROM users WHERE status IN ('active', 'pending')",
  "message": "Query executed successfully in Snowflake"
}
```

### Example 2: Complex Request → Snowflake Execution
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
**Snowflake Response:**
```json
{
  "query_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "sql": "SELECT id, name, price FROM products WHERE category = 'electronics' AND price >= '100' ORDER BY name, price DESC LIMIT 50",
  "message": "Query executed successfully in Snowflake"
}
```

## 🔧 QueryBuilder: The Heart of SQL Generation & Snowflake Execution

The `QueryBuilder` is the central component that:
1. **Receives** validated request data from FastAPI routes
2. **Creates** appropriate QueryDefinition instance
3. **Orchestrates** the strategy-based SQL building process  
4. **Executes** generated SQL against Snowflake database
5. **Returns** Snowflake query_id and SQL in formatted response

```python
# This is what happens when you send an HTTP request:
def build_query(self, query_parameters: QueryParameters) -> SimpleQueryResponse:
    # 1. Create query definition for this request type
    query_definition = SimpleQueryDefinition()
    
    # 2. Process request parameters through strategies  
    sql = query_definition.build_query(query_parameters)
    
    # 3. Execute SQL against Snowflake (in production)
    # snowflake_query_id = execute_in_snowflake(sql)
    
    # 4. Generate tracking ID and format response
    query_id = str(uuid.uuid4())  # In production: snowflake_query_id
    
    # 5. Return Snowflake query_id and SQL to client
    return SimpleQueryResponse(query_id=query_id, sql=sql, message="success")
```

## 🚀 Production vs Simplified Version

### **Production (Full Snowflake Integration):**
- ✅ Executes SQL against real Snowflake databases
- ✅ Returns actual Snowflake query_id from execution
- ✅ Results stored in Snowflake, retrievable by query_id
- ✅ Full error handling for Snowflake connection issues

### **Simplified (Demo Architecture):**
- ✅ Same architectural patterns as production
- ✅ Generates identical SQL queries
- ✅ Uses dummy Snowflake credentials (.env file)
- ⚠️ Generates UUID instead of real Snowflake query_id
- ⚠️ SQL not actually executed (demonstration only)

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

