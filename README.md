# Alerta Belém

Aplicação web responsiva para alertas colaborativos de alagamento em Belém, Pará. O sistema permite que usuários consultem ocorrências no mapa, publiquem novos alertas com localização e foto, confirmem informações e recebam avisos dentro do aplicativo.

## Sumário

- [Objetivo](#objetivo)
- [Funcionalidades](#funcionalidades)
- [Tecnologias](#tecnologias)
- [Arquitetura](#arquitetura)
- [Como executar](#como-executar)
- [API REST](#api-rest)
- [Banco de dados](#banco-de-dados)
- [Mapa e localização](#mapa-e-localização)
- [Upload de imagens](#upload-de-imagens)
- [Estrutura de pastas](#estrutura-de-pastas)
- [Escopo da V1](#escopo-da-v1)

## Objetivo

Durante períodos de chuva, moradores, motoristas, motociclistas e pedestres precisam identificar rapidamente áreas alagadas. O Alerta Belém centraliza relatos colaborativos para apoiar decisões de deslocamento mais seguro.

## Funcionalidades

- Cadastro e login com e-mail, senha e token JWT.
- Perfil conectado aos dados da conta criada no banco.
- Mapa de Belém com ocorrências classificadas por nível de risco.
- Localização do usuário via Geolocation API do navegador, mediante permissão.
- Criação de alertas com coordenadas, nível, horário, impactos e descrição.
- Anexo de fotos JPG, PNG ou WEBP de até 5 MB.
- Listagem, filtros e detalhes de ocorrências.
- Confirmação colaborativa de ocorrências.
- Atualização do status de uma ocorrência para ativa ou resolvida.
- Central de notificações e marcador de não lidas.
- Documentação interativa da API com Swagger/OpenAPI.

## Tecnologias

| Camada | Tecnologia | Papel no projeto |
| --- | --- | --- |
| Front-end | React, TypeScript e Vite | Interface responsiva e integração com a API |
| Mapa | React Leaflet e OpenStreetMap | Mapa, marcadores e visualização de áreas de risco |
| Localização | Geolocation API | Captura a posição autorizada pelo usuário |
| Back-end | FastAPI | API REST, rotas, validações e Swagger automático |
| Dados | SQLAlchemy | Mapeamento das tabelas e consultas ao banco |
| Validação | Pydantic | Validação de e-mail, senha, coordenadas e campos da API |
| Autenticação | JWT, Passlib e Bcrypt | Sessão autenticada e armazenamento seguro de senhas |
| Banco local | SQLite | Execução local sem Docker |
| Banco em container | PostgreSQL e PostGIS | Ambiente previsto para Docker Compose |
| Upload | FastAPI `UploadFile` | Armazenamento local de fotos em desenvolvimento |

## Arquitetura

```text
Navegador
  React + Vite + React Leaflet
           |
           | HTTP JSON + Bearer JWT
           v
FastAPI
  rotas | validação | regras | upload
           |
           v
SQLAlchemy
  SQLite local ou PostgreSQL/PostGIS com Docker
```

## Como executar

### Pré-requisitos

- Node.js 18 ou superior.
- Python 3.12 ou superior.
- Docker Desktop, apenas para a opção PostgreSQL/PostGIS.

### 1. Back-end sem Docker

Abra um terminal na raiz do projeto e execute:

```powershell
cd backend
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install --upgrade -r requirements.txt
.\.venv\Scripts\python.exe -m uvicorn app.main:app --reload
```

Mantenha o terminal aberto. A API estará disponível em `http://localhost:8000`.

### 2. Front-end

Abra outro terminal na raiz do projeto e execute:

```powershell
cd frontend
npm install
npm run dev
```

Abra a URL exibida pelo Vite, normalmente `http://localhost:5173`.

### 3. Executar com Docker

Se o Docker Desktop estiver instalado e aberto:

```powershell
docker compose up --build
```

Em outro terminal, inicie o front-end com os comandos da seção anterior.

> Sem Docker, o back-end usa SQLite automaticamente. Com Docker, o arquivo `docker-compose.yml` prepara PostgreSQL com PostGIS.

## API REST

Base local: `http://localhost:8000/api/v1`  
Documentação Swagger: `http://localhost:8000/docs`

As rotas protegidas exigem o cabeçalho:

```http
Authorization: Bearer SEU_TOKEN_JWT
```

| Método | Rota | Descrição | Protegida |
| --- | --- | --- | --- |
| `GET` | `/health` | Verifica se a API está disponível | Não |
| `POST` | `/auth/register` | Cria uma conta e retorna um token | Não |
| `POST` | `/auth/login` | Autentica o usuário | Não |
| `GET` | `/me` | Retorna os dados da conta autenticada | Sim |
| `GET` | `/me/profile` | Retorna perfil e estatísticas do usuário | Sim |
| `PATCH` | `/me` | Atualiza nome e e-mail do perfil | Sim |
| `GET` | `/occurrences` | Lista ocorrências | Não |
| `GET` | `/occurrences/{id}` | Exibe uma ocorrência | Não |
| `POST` | `/occurrences` | Cria uma ocorrência | Sim |
| `POST` | `/occurrences/{id}/confirm` | Confirma uma ocorrência | Sim |
| `PATCH` | `/occurrences/{id}` | Atualiza o status da ocorrência | Sim |
| `POST` | `/uploads` | Envia uma imagem de ocorrência | Sim |
| `GET` | `/notifications` | Lista notificações | Sim |
| `PATCH` | `/notifications/{id}/read` | Marca uma notificação como lida | Sim |

## Banco de dados

O back-end cria as tabelas no primeiro início:

| Tabela | Finalidade |
| --- | --- |
| `users` | Contas, e-mails e senhas com hash |
| `occurrences` | Alertas de alagamento, nível de risco, localização, foto e status |
| `notifications` | Avisos exibidos aos usuários |
| `confirmations` | Confirmações de usuários por ocorrência |

Na execução local, os dados são armazenados em `backend/alerta_belem.db`. As coordenadas são guardadas em latitude e longitude. A configuração Docker prepara PostgreSQL/PostGIS para evolução geoespacial do projeto.

## Mapa e localização

O mapa usa OpenStreetMap através do React Leaflet. Os marcadores exibem o risco de cada ocorrência:

| Nível | Cor | Significado |
| --- | --- | --- |
| Atenção | Amarelo | Acúmulo leve de água |
| Moderado | Laranja | Dificuldade de passagem |
| Alto | Vermelho | Via parcialmente bloqueada |
| Grave | Vermelho escuro | Via intransitável ou risco real |

O navegador pede permissão para usar a localização. Se o usuário negar, o aplicativo mantém o mapa disponível, mas não consegue centralizar a posição exata automaticamente.

## Upload de imagens

O formulário de relato aceita:

- JPG/JPEG
- PNG
- WEBP
- Arquivos de até 5 MB

Em desenvolvimento, o back-end armazena as imagens em `backend/uploads/` e as disponibiliza pela rota `/uploads/{arquivo}`. Para produção, o projeto pode migrar esse serviço para Cloudinary ou outro armazenamento de objetos.

## Estrutura de pastas

```text
.
├── frontend/                 # Aplicação React/Vite
│   └── src/
│       ├── api.ts            # Cliente da API
│       ├── main.tsx          # Telas e navegação
│       ├── MapView.tsx       # Mapa e marcadores
│       └── styles.css        # Estilos da interface
├── backend/                  # API FastAPI
│   ├── app/
│   │   ├── main.py           # Rotas da API
│   │   ├── models.py         # Modelos SQLAlchemy
│   │   ├── schemas.py        # Schemas Pydantic
│   │   └── security.py       # JWT e hash de senha
│   ├── uploads/              # Fotos locais em desenvolvimento
│   └── requirements.txt
├── docs/                     # Design, componentes e fluxos
├── docker-compose.yml        # PostgreSQL/PostGIS com Docker
└── README.md
```

## Escopo da V1

### Incluído

Cadastro, autenticação, GPS, mapa de ocorrências, relatos com foto, níveis de risco, impactos em trânsito/pedestres/imóveis, confirmações, resolução de ocorrência e notificações internas.

### Não incluído

- Previsão meteorológica própria.
- Monitoramento por câmeras ou sensores físicos.
- Acionamento de equipes de resgate.
- Integração direta com órgãos públicos.
- Firebase Cloud Messaging e Cloudinary. Esses serviços ficam como evolução futura.

## Equipe

- Ramiro Augusto Miranda Neto - 04182362
- Rebeca Matos - 04174723
- Sabrina Maués da Silva - 04181448

## Documentação complementar

- [Tokens de design](docs/design-tokens.md)
- [Componentes reutilizáveis](docs/componentes.md)
- [Fluxo de navegação](docs/fluxo-navegacao.md)
- [Contrato de telas e API](docs/telas-e-api.md)
