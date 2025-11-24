# Letrar IA - Backend

Backend da aplicação Letrar IA construído com FastAPI e PostgreSQL.

## Tecnologias

- **Python 3.11+**
- **FastAPI** - Framework web assíncrono
- **SQLAlchemy 2.x** - ORM com suporte async
- **Alembic** - Gerenciamento de migrations
- **PostgreSQL** - Banco de dados relacional
- **Pydantic V2** - Validação de dados e configurações
- **python-jose** - Geração e validação de tokens JWT
- **bcrypt** - Criptografia de senhas
- **asyncpg** - Driver PostgreSQL assíncrono

## Arquitetura

O backend utiliza uma **Arquitetura em Camadas (Layered Architecture)** seguindo os princípios de **Clean Architecture** e **SOLID**, organizada em camadas bem definidas:

### Camadas da Arquitetura

```
┌─────────────────────────────────────┐
│        API Routes (Routes)          │  ← Camada de Apresentação
│     (Endpoints HTTP/REST)           │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│         Services                    │  ← Camada de Lógica de Negócio
│   (Regras de negócio, orquestração) │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│      Repositories                   │  ← Camada de Acesso a Dados
│   (Operações com banco de dados)    │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│         Models                      │  ← Camada de Dados
│   (Entidades SQLAlchemy)            │
└─────────────────────────────────────┘
```

### Fluxo de Dados

1. **Routes** (`api/routes/`) - Recebe requisições HTTP, valida entrada com Schemas Pydantic
2. **Services** (`services/`) - Contém a lógica de negócio, orquestra operações entre repositories
3. **Repositories** (`repositories/`) - Abstrai acesso ao banco de dados, realiza queries SQLAlchemy
4. **Models** (`models/`) - Define entidades do banco de dados (ORM)

## Padrões de Implementação

### 1. Repository Pattern

Cada entidade tem seu próprio repository que encapsula todas as operações de acesso a dados:

```python
class UserRepository:
    def __init__(self, session: AsyncSession):
        self.session = session
    
    async def get_by_email(self, email: str) -> Optional[User]:
        # Implementação específica de busca por email
```

**Benefícios:**
- Isola a lógica de acesso a dados
- Facilita testes (pode criar mocks)
- Permite trocar implementação sem afetar serviços

### 2. Service Layer Pattern

Serviços contêm a lógica de negócio e orquestram operações:

```python
class AuthService:
    def __init__(self, session: AsyncSession):
        self.user_repository = UserRepository(session)
    
    async def authenticate_user(self, email: str, password: str):
        # Lógica de autenticação
```

**Benefícios:**
- Separa lógica de negócio das rotas
- Facilita reutilização de código
- Torna testes mais simples

### 3. Dependency Injection

FastAPI utiliza dependency injection nativo através de `Depends()`:

```python
async def login(
    login_data: LoginRequest,
    db: AsyncSession = Depends(get_db),
):
    # db é injetado automaticamente
```

### 4. Schema Pattern (DTO)

Schemas Pydantic validam e transformam dados:

- **Request Schemas** - Validam dados de entrada
- **Response Schemas** - Formatam dados de saída
- **Separação entre Models (ORM) e Schemas (API)**

## Princípios de Design

### Single Responsibility Principle (SRP)
- Cada classe tem uma única responsabilidade
- Routes: apenas recebem requisições e retornam respostas
- Services: apenas contêm lógica de negócio
- Repositories: apenas acessam dados

### Dependency Inversion Principle (DIP)
- Camadas superiores dependem de abstrações (interfaces)
- Services dependem de Repositories, não diretamente de Models
- Facilita testes e mudanças de implementação

### Separation of Concerns
- Lógica de negócio separada de acesso a dados
- Validação separada de transformação
- Autenticação isolada em utils

## Exemplo de Fluxo Completo

Vamos seguir um exemplo de **Login** para entender o fluxo:

```
1. Cliente HTTP → POST /auth/login {email, password}
   ↓
2. Route (auth.py)
   - Valida entrada com LoginRequest (Pydantic)
   - Injeta dependência get_db()
   ↓
3. Service (auth_service.py)
   - Cria UserRepository
   - Chama repository.get_by_email()
   - Verifica senha com verify_password()
   - Gera token JWT com create_access_token()
   ↓
4. Repository (user_repository.py)
   - Executa query SQLAlchemy async
   - Retorna User model ou None
   ↓
5. Model (user.py)
   - Retorna instância de User (ORM)
   ↓
6. Service retorna dict com token e dados do usuário
   ↓
7. Route retorna resposta JSON
```

## Convenções de Código

### Nomenclatura

- **Models**: PascalCase (`User`, `Student`, `Trail`)
- **Repositories**: PascalCase + "Repository" (`UserRepository`)
- **Services**: PascalCase + "Service" (`AuthService`)
- **Routes/Functions**: snake_case (`get_current_user`, `authenticate_user`)
- **Schemas**: PascalCase (`LoginRequest`, `Token`, `UserResponse`)
- 
### Imports

- Imports absolutos: `from app.models.user import User`
- Evitar imports circulares
- Agrupar imports: stdlib, third-party, local

## Configuração Inicial

### Usando Docker (Recomendado)

Ver instruções completas no README.md principal do projeto.

### Desenvolvimento Local

1. Criar ambiente virtual:
```bash
python -m venv venv
source venv/bin/activate  # Linux/Mac
# ou
venv\Scripts\activate  # Windows
```

2. Instalar dependências:
```bash
pip install -r requirements.txt
```

3. Copiar arquivo de ambiente:
```bash
cp env.example .env
```

4. Configurar variáveis de ambiente no arquivo `.env`:
   - `DATABASE_URL`: URL do PostgreSQL
   - `SECRET_KEY`: Chave secreta para JWT
   - `CORS_ORIGINS`: Lista de origens permitidas

5. Aplicar migrations:
```bash
alembic upgrade head
```

6. Executar seeder (opcional):
```bash
python run_seed.py
```

7. Executar aplicação:
```bash
uvicorn app.main:app --reload
```

A API estará disponível em `http://localhost:8000`

```
