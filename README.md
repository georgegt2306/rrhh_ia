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
WHERE numero_identificacion = '0942097601';
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

    MAX(CASE WHEN cfg_arch.id = 95  THEN 'TIENE' ELSE 'NO TIENE' END) AS "Hoja de vida",
    MAX(CASE WHEN cfg_arch.id = 101 THEN 'TIENE' ELSE 'NO TIENE' END) AS "Antecedentes penales",
    MAX(CASE WHEN cfg_arch.id = 97  THEN 'TIENE' ELSE 'NO TIENE' END) AS "Cédula de identidad",
    MAX(CASE WHEN cfg_arch.id = 98  THEN 'TIENE' ELSE 'NO TIENE' END) AS "Certificado de votación",
    MAX(CASE WHEN cfg_arch.id = 99  THEN 'TIENE' ELSE 'NO TIENE' END) AS "Certificado Laboral",
    MAX(CASE WHEN cfg_arch.id = 36  THEN 'TIENE' ELSE 'NO TIENE' END) AS "Título Bachiller",
    MAX(CASE WHEN cfg_arch.id = 21  THEN 'TIENE' ELSE 'NO TIENE' END) AS "Certificado Agente de Seguridad"
FROM personas per
...
```

---

## ✅ Resultado
Una fila por colaborador con estado TIENE / NO TIENE para cada documento.

---

📌 Última actualización: 2025
