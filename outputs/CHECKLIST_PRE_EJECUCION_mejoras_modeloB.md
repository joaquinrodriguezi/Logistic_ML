# ✅ CHECKLIST PRE-EJECUCIÓN - mejoras_de_modeloB.ipynb

## ANTES DE EJECUTAR EL NOTEBOOK, VERIFICAR:

### 🔴 CRÍTICO - Data Leakage

- [ ] **processing_time NO está en features**
  - Buscar en código: `'processing_time'` → debe estar en `forbidden_features`, NO en `numeric_features`
  - Assert debe existir: `assert 'processing_time' not in all_features`

- [ ] **LabelEncoder hace fit SOLO en train**
  - Buscar: `le.fit(X_train_raw[col])` → ✅ CORRECTO
  - NO debe aparecer: `pd.concat([X_train, X_test])` en encoding

- [ ] **Split temporal por fecha fija**
  - Buscar: `CUTOFF_DATE = pd.Timestamp('2017-04-25')`
  - Debe existir assert: `train.max() < test.min()`

### 🟡 IMPORTANTE - Validaciones

- [ ] **Variables prohibidas documentadas**
  - Lista `forbidden_features` debe incluir:
    - Days_for_shipping_(real)
    - Delivery_Status
    - Late_delivery_risk
    - processing_time
    - delay_days
    - delay_pos
    - y_cls

- [ ] **Chequeo de duplicados implementado**
  - Debe reportar duplicados por Order_Id + shipping_date

- [ ] **Etapa 1 y 2 correctamente separadas**
  - Etapa 1: entrena con TODO train (y_cls)
  - Etapa 2: entrena SOLO con positivos (delay_pos > 0)

### 🟢 OPCIONAL - Mejoras

- [ ] **Reproducibilidad**
  - random_state=42 en modelos
  - Prints con logs detallados
  - Seed fija si se usa numpy random

- [ ] **Documentación**
  - Celdas markdown explican correcciones
  - Comentarios en código sobre anti-leakage

---

## AL EJECUTAR EL NOTEBOOK, OBSERVAR:

### 📊 Outputs Esperados

1. **Print de exclusión de processing_time**
   ```
   ⚠️ NOTA: processing_time fue EXCLUIDO (riesgo de leakage...)
   ```

2. **Confirmación de asserts**
   ```
   ✓ Assert: processing_time correctamente excluido
   ✓ Verificación anti-leakage: PASS
   ```

3. **Categorías desconocidas reportadas** (si existen)
   ```
   ⚠️ Order_City: 15 categorías desconocidas en TEST (codificadas como -1)
   ```

4. **Split temporal sin overlap**
   ```
   ✓ Split temporal completado (sin overlap verificado)
   ```

### 📉 Cambios Esperados en Métricas

**COMPARADO CON MODELO ANTERIOR (si tenía leakage):**

| Métrica | Antes (con leak) | Después (sin leak) | Cambio |
|---------|------------------|-------------------|--------|
| PR-AUC (Etapa 1) | ~0.90-0.95 | ~0.70-0.80 | ⬇️ -10-20% |
| MAE_pos (Etapa 2) | ~0.5-1.5 días | ~2.0-3.5 días | ⬆️ +50-150% |
| delay_expected MAE | ~0.8-1.2 días | ~1.5-2.5 días | ⬆️ +50-100% |

**INTERPRETACIÓN:**
- ⚠️ Si las métricas EMPEORAN → ✅ CORRECTO (se eliminó leakage)
- ✅ Si las métricas quedan aceptables para negocio → Proceder
- ❌ Si las métricas empeoran mucho → Necesita feature engineering adicional

---

## DESPUÉS DE EJECUTAR, VALIDAR:

### ✅ Archivos Generados

- [ ] `mejoras_modeloB_predicciones_test.csv` creado
- [ ] `mejoras_modeloB_metricas.csv` creado  
- [ ] `mejoras_modeloB_comparacion_vs_anterior.txt` creado

### ✅ Métricas Razonables

- [ ] PR-AUC clasificación: > 0.65 (mínimo viable)
- [ ] MAE_pos regresión: < 4 días (error aceptable)
- [ ] Mejora vs Baseline 1: > 10% en MAE (justifica complejidad)

### ⚠️ Red Flags (Indicadores de Problemas)

- [ ] ❌ PR-AUC > 0.95 → Puede haber leakage residual
- [ ] ❌ MAE_pos < 1 día → Sospechosamente bajo, revisar
- [ ] ❌ Categorías desconocidas > 10% → Problema de encoding
- [ ] ❌ Overlap temporal detectado → Split mal implementado

---

## COMANDOS ÚTILES PARA DEBUGGING

### Verificar leakage en features (manual)
```python
# Ejecutar en celda separada DESPUÉS de definir all_features
print("Features activas:", all_features)
print("\n¿processing_time presente?", 'processing_time' in all_features)
print("¿Days_for_shipping_real presente?", 'Days_for_shipping_(real)' in all_features)

# Verificar correlación sospechosa (si processing_time existiera en df)
if 'processing_time' in train.columns and 'Days_for_shipping_(real)' in train.columns:
    corr = train[['processing_time', 'Days_for_shipping_(real)']].corr().iloc[0,1]
    print(f"\nCorrelación processing_time vs real: {corr:.3f}")
    if abs(corr) > 0.8:
        print("⚠️ ALERTA: Correlación alta detectada (>0.8) → riesgo de leakage")
```

### Verificar split temporal (manual)
```python
# Ejecutar DESPUÉS del split
print("Train max date:", train['shipping_date_(DateOrders)'].max())
print("Test min date:", test['shipping_date_(DateOrders)'].min())
print("Gap (días):", (test['shipping_date_(DateOrders)'].min() - train['shipping_date_(DateOrders)'].max()).days)

# Debe ser >= 0 (sin overlap)
```

### Verificar encoding (manual)
```python
# Ver si hay categorías -1 en test (desconocidas)
for col in categorical_features:
    unknown_count = (test[col] == -1).sum()
    if unknown_count > 0:
        print(f"{col}: {unknown_count} desconocidas ({unknown_count/len(test)*100:.1f}%)")
```

---

## CONTACTO PARA ISSUES

Si encuentra problemas NO cubiertos por este checklist:

1. **Verificar que aplicó todas las correcciones** (buscar en código)
2. **Revisar archivo**: `AUDITORIA_ANTI_LEAKAGE_mejoras_modeloB.txt`
3. **Ejecutar comandos de debugging** (arriba)
4. **Documentar el error** con:
   - Output exacto del error
   - Celda donde ocurre
   - Screenshots de variables relevantes

---

**Fecha:** 15 de Marzo, 2026  
**Versión:** 1.0 (Post-Auditoría Anti-Leakage)  
**Status:** ✅ Listo para ejecución
