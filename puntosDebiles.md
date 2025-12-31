# Limitaciones del enfoque Z3 en ÆTHER

> Análisis crítico del claim de "Gobernanza de IA Demostrable"

---

## 1. El cuello de botella de la traducción

El sistema asume que puedes traducir correctamente `intención del LLM → lógica formal`. Esto es **el problema más difícil** y el documento lo trata como trivial.

```python
# El LLM dice: "Podemos hacer una excepción para este cliente VIP"
# ¿Cómo traduces "excepción" a lógica formal?
# ¿Qué significa "VIP"? ¿Es un campo booleano? ¿Un score > X?
```

Si la traducción es incorrecta, Z3 valida algo que no representa la acción real. **Basura entra, basura sale.**

---

## 2. No todas las reglas son formalizables

Algunas reglas de negocio son inherentemente ambiguas:

| Regla | Problema de formalización |
|-------|---------------------------|
| "No hacer promesas que no podamos cumplir" | Requiere conocer el estado futuro |
| "Responder de forma apropiada al contexto" | "Apropiado" es subjetivo |
| "Priorizar clientes estratégicos" | "Estratégico" cambia según quién preguntes |
| "Actuar de buena fe" | Concepto legal, no lógico |

Z3 solo puede verificar lo que puedes expresar en lógica de primer orden.

---

## 3. Explosión de complejidad

SMT solving es **NP-hard** en el caso general. Con muchas restricciones interconectadas:

```python
# 10 reglas simples → milisegundos
# 1000 reglas con dependencias → potencialmente minutos/horas
# Reglas con cuantificadores anidados → puede no terminar

from z3 import *
s = Solver()
s.set("timeout", 5000)  # Necesitas timeouts en producción

# Si el timeout se dispara, ¿qué haces?
# ¿Bloqueas por defecto? ¿Permites por defecto?
# Ambas opciones tienen riesgos
```

---

## 4. Completitud de las reglas (el problema real)

Z3 **solo verifica contra las reglas que le das**. No inventa reglas faltantes.

El caso Air Canada que menciona el documento: el problema no fue que el chatbot violó una regla —fue que **la regla no existía** en el sistema. Z3 no habría ayudado si nadie codificó `Politica_Reembolso_Duelo`.

```python
# Tu "Constitución Ética" tiene 500 reglas
# ¿Cómo sabes que no te faltan 50 más críticas?
# Z3 no puede decirte: "Oye, deberías tener una regla sobre X"
```

---

## 5. El specification gap

Lo que escribes en lógica formal puede no ser lo que querías decir:

```python
# Intención: "Solo directores pueden aprobar gastos > 50k"
# Lo que escribiste:
regla = Implies(gasto > 50000, aprobador.nivel == "Director")

# Bug: No verificaste que el aprobador sea del mismo departamento
# Bug: No verificaste que el aprobador no sea el solicitante
# Bug: No verificaste que el aprobador esté activo en el sistema
```

---

## 6. Mantenimiento y drift

Las reglas formales necesitan actualizarse constantemente:

- Cambia una política → actualizar reglas
- Nueva regulación → añadir reglas
- Caso edge descubierto → parchar reglas

En la práctica, las reglas formales **divergen de la realidad** con el tiempo. Nadie tiene un equipo dedicado a mantener especificaciones Z3 sincronizadas con el negocio.

---

## 7. Lo que Z3 NO previene

El documento implica que Z3 resuelve los problemas de IA empresarial. No previene:

| Problema | ¿Z3 ayuda? |
|----------|------------|
| Alucinaciones factuales | ❌ No (el LLM puede inventar datos que pasan validación) |
| Sesgos en contenido generado | ❌ No (difícil formalizar "sesgo") |
| Prompt injection sofisticado | ⚠️ Parcial (si el ataque genera acciones válidas según las reglas) |
| Información correcta pero desactualizada | ❌ No |
| Tono inapropiado | ❌ No |

---

## 8. Falsos positivos y fricción operacional

Z3 puede bloquear acciones legítimas:

```python
# Regla: No ofrecer descuentos > 20%
s.add(descuento <= 0.20)

# Situación: Cliente amenaza con irse, vale 2M€/año
# Comercial quiere ofrecer 25% para retenerlo
# Z3: BLOQUEADO

# El override manual existe, pero añade fricción
# Si hay muchos falsos positivos, la gente deja de confiar en el sistema
```

---

## Evaluación general

El enfoque de Z3 es **valioso pero sobrevalorado** en el documento. Es una capa de defensa útil para reglas claras y binarias (límites de gasto, permisos, compliance regulatorio específico).

Pero el claim de "inmunidad matemática" es exagerado. La seguridad real depende de:

1. Calidad de la traducción LLM → lógica
2. Completitud de las reglas
3. Mantenimiento continuo
4. Combinación con otras técnicas (testing, monitoring, human-in-the-loop)

---

## TODO: Mitigaciones

- [ ] Estrategias para mejorar la traducción LLM → lógica formal
- [ ] Técnicas para descubrir reglas faltantes
- [ ] Arquitectura híbrida Z3 + otras capas de defensa
- [ ] Proceso de mantenimiento de especificaciones

---

*Última actualización: Diciembre 2025*
