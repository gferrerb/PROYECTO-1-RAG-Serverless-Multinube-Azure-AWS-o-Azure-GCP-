# Proyecto RAG Multicloud Serverless (AWS + GCP + OpenAI)
**Versión:** 1.0  
**Autor:** Gustavo Ferrer  
**Arquitectura:** Serverless – Multinube  
**Clouds utilizadas:** AWS + GCP  
**LLM:** OpenAI (API pública)  
**Vector Store:** AlloyDB + pgvector  
**Orquestación:** AWS Lambda + GCP Cloud Run  
**Alcance final del laboratorio:** ✔ Completado

---

# 1. Resumen Ejecutivo

Este proyecto implementa una plataforma **RAG Multicloud Serverless** utilizando:

- **AWS S3** como storage de documentos
- **AWS Lambda** como disparador de ingesta
- **GCP Cloud Run** como microservicio de chunking y embeddings
- **GCP AlloyDB (pgvector)** como vector store
- **OpenAI API** como proveedor del modelo de embeddings
- **Arquitectura serverless** sin uso de VMs ni Kubernetes

El pipeline demuestra cómo una arquitectura de IA moderna puede operar en **dos nubes** y coordinarse de forma eficiente sin infraestructura tradicional.

### Objetivo del laboratorio  
Construir un pipeline RAG mínimo funcional con al menos dos nubes, demostrando:

- Integración cross-cloud
- Diseño serverless
- Ingesta documental automática
- Chunking + embeddings
- Indexación en vector store
- Preparación para retrieval + LLM inference

---

# 2. Alcance Final Implementado

| Componente | Estado | Descripción |
|-----------|--------|-------------|
| AWS S3 | ✔ | Almacena documentos fuente enviados por el usuario |
| AWS Lambda | ✔ | Detecta subida de archivos y llama al chunker en GCP |
| GCP Cloud Run – chunker | ✔ | Divide documentos, genera embeddings y escribe en AlloyDB |
| GCP AlloyDB (pgvector) | ✔ | Vector store con embeddings |
| OpenAI embeddings API | ✔ | Generación de embeddings |
| Retrieval | ✖ (solo diseño) | Se documenta pero no se implementa |
| Azure | ✖ | Eliminado del alcance final del laboratorio |

---

# 3. Arquitectura General (Nivel Alto)

+-------------+ +----------------+ +------------------+
| Usuario | upload | AWS S3 Bucket | event | AWS Lambda |
| +------------->+ (raw docs) +-----------> + ingest-trigger |
+-------------+ +----------------+ +------------------+
|
| HTTPS POST
v
+-----------------------+
| GCP Cloud Run |
| chunker + embeddings |
+-----------+-----------+
|
| psycopg + pgvector
v
+---------------------+
| GCP AlloyDB |
| Vector Store |
+---------------------+


---

# 4. Pipeline Completo Implementado

### 1️⃣ **Upload a AWS S3**
El usuario sube un archivo `.txt` a un bucket configurado.

### 2️⃣ **AWS Lambda — Trigger automático**
Lambda recibe el evento S3 (`ObjectCreated:Put`):

- Obtiene `bucket`, `key`, `doc_id`
- Llama por POST al endpoint público de Cloud Run:
  
```json
POST https://chunker-…run.app/ingest
{
  "mode": "s3",
  "doc_id": "Flaxius",
  "s3_bucket": "rag-lab-gferrer",
  "s3_key": "Flaxius.txt"
}
