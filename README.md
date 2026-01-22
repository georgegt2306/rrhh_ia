# 📘 Integración RRHH – Ranking, Puntajes, Pruebas IESSE y Validación Documental

Este documento describe el flujo de integración de información de agentes y colaboradores, combinando ranking mensual, puntajes, pruebas externas y validación de archivos obligatorios de RRHH.

---

## 🧩 Flujo General

1. Obtener ranking mensual de agentes  
2. Identificar agentes por cédula / usuario  
3. Consultar puntajes internos (Katuk)  
4. Consultar resultados de pruebas IESSE  
5. Validar documentos obligatorios cargados  
6. Consolidar información para toma de decisiones  

---

## 1️⃣ Obtención de ranking mensual de agentes

### Endpoint
POST http://172.18.1.3:43080/sgi.api.in/ws/usuario/ranking-mensual

### BEARER TOKEN
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJPbmxpbmUgSldUIEJ1aWxkZXIiLCJpYXQiOjE2ODU2NDYyMzAsImV4cCI6MTcxNzE4MjIzMCwiYXVkIjoid3d3LmV4YW1wbGUuY29tIiwic3ViIjoianJvY2tldEBleGFtcGxlLmNvbSIsIkdpdmVuTmFtZSI6IkpvaG5ueSIsIlN1cm5hbWUiOiJSb2NrZXQiLCJFbWFpbCI6Impyb2NrZXRAZXhhbXBsZS5jb20iLCJSb2xlIjpbIk1hbmFnZXIiLCJQcm9qZWN0IEFkbWluaXN0cmF0b3IiXX0.Z1_OZGsnPUT3_7IsaaELFR9Xq_9VvncYZlCdw8_j-vw

### Body
```json
{
  "nemotecnico": "CM-EC",
  "mes": 12,
  "anio": 2025,
  "top": 10
}
```

### Response (ejemplo)
```json
[
  {
    "apellido": "Valero Gómez",
    "cedula": "0942097601",
    "nombre": "Aldo Steven",
    "posicion": 1,
    "usuarioId": 8431,
    "valor": 10.0
  }
]
```

---

## 2️⃣ Consulta de puntajes – Katuk

### Obtener user_id por cédula
```sql
SELECT user_id
FROM rrhh_aspirante_postulantes_katuk
WHERE numero_identificacion = '<USER_ID>';
```

### Obtener puntajes
```sql
SELECT title, puntaje, ponderacion_aprobacion
FROM rrhh_aspirante_postulantes_katuk_det
WHERE user_id = '<USER_ID>';
```

---

## 3️⃣ Consulta de pruebas IESSE

### Endpoint
POST https://apps.iesse.ec/iesse/api/listarSecuenciaporDocumento.php

### Body
```json
{
  "documento": ["0942097601", "0942234295"]
}
```

### Response (ejemplo)
```json
{
  "documento": "0942097601",
  "secuencias": [
    {
      "secuencia": "Agente de Seguridad",
      "estado": "Culminado",
      "nota": 8.46
    }
  ]
}
```

---

## 4️⃣ Validación de documentos obligatorios

### Documentos evaluados
- Hoja de vida  
- Antecedentes penales  
- Cédula de identidad  
- Certificado de votación  
- Certificado laboral  
- Título de bachiller  
- Certificado Agente de Seguridad  

### Query SQL
```sql
SELECT
    per.numero_identidad,
    per.nombre_primero || ' ' || per.nombre_segundo || ' ' ||
    per.apellido_primero || ' ' || per.apellido_segundo AS nombres,

    COALESCE(
        MAX(CASE WHEN cfg_arch.id = 95 THEN rrhh_ar.server_path || rrhh_ar.filename END),
        'NO TIENE'
    ) AS "Hoja de Vida",

    COALESCE(
        MAX(CASE WHEN cfg_arch.id = 101 THEN rrhh_ar.server_path || rrhh_ar.filename END),
        'NO TIENE'
    ) AS "Antecedentes Penales",

    COALESCE(
        MAX(CASE WHEN cfg_arch.id = 97 THEN rrhh_ar.server_path || rrhh_ar.filename END),
        'NO TIENE'
    ) AS "Cédula",

    COALESCE(
        MAX(CASE WHEN cfg_arch.id = 98 THEN rrhh_ar.server_path || rrhh_ar.filename END),
        'NO TIENE'
    ) AS "Certificado de Votación",

    COALESCE(
        MAX(CASE WHEN cfg_arch.id = 99 THEN rrhh_ar.server_path || rrhh_ar.filename END),
        'NO TIENE'
    ) AS "Certificado Laboral",

    COALESCE(
        MAX(CASE WHEN cfg_arch.id = 36 THEN rrhh_ar.server_path || rrhh_ar.filename END),
        'NO TIENE'
    ) AS "Título de Bachiller",

    COALESCE(
        MAX(CASE WHEN cfg_arch.id = 21 THEN rrhh_ar.server_path || rrhh_ar.filename END),
        'NO TIENE'
    ) AS "Certificado de curso"

FROM personas per
JOIN persona_tiempo_laborado ptl
    ON ptl.persona_id = per.id
    AND ptl.estado = 'A'
    AND ptl.empresa_id = 1

LEFT JOIN rrhh_archivos rrhh_ar
    ON rrhh_ar.persona_tiempo_laborado_id = ptl.id
    AND rrhh_ar.estado = 'A'

LEFT JOIN cfg_tipos_archivos cfg_arch
    ON cfg_arch.id = rrhh_ar.tipo_archivo_id
    AND cfg_arch.id IN (95,101,97,98,99,36,21)

WHERE ptl.id IN (7324)

GROUP BY
    per.numero_identidad,
    per.nombre_primero,
    per.nombre_segundo,
    per.apellido_primero,
    per.apellido_segundo

ORDER BY per.numero_identidad DESC;

```

---

## ✅ Resultado
Una fila por colaborador si tiene la ruta o no para cada documento.
<img width="653" height="79" alt="image" src="https://github.com/user-attachments/assets/63252a6c-6fd4-4fee-8d33-fdc5f6d3dd19" />

---

📌 Última actualización: 2025
