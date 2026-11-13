# Derivación automática de tickets de soporte

Proyecto Integrador · Desarrollo de Soluciones Inteligentes (02DSI243061) · UBO · 2026_S2
Equipo: Vicente Troncoso y Diego Silva · Entrega EVAL-01, 13-11-2026

## Objetivo

Los tickets de soporte llegan mezclados y una persona debe leerlos para decidir a qué categoría van. El prototipo lee el texto de un ticket y sugiere la categoría (11 posibles). Si la confianza del modelo es menor a 0,6, no deriva y deja el ticket para revisión manual.

## Contenido

| Archivo | Para qué sirve |
|---|---|
| `Derivacion_Tickets_Colab.ipynb` | Todo el flujo: datos, comparación de técnicas, modelo, umbral, aplicación, validación y casos límite |
| `EVAL01_Evidencia_Tecnica.docx` | Informe de evidencia con resultados, riesgos y declaración de uso de IA |
| `README.md` | Esta documentación |

## Cómo ejecutar

1. Descargar el CSV `Bitext_Sample_Customer_Support_Training_Dataset_27K_responses-v11.csv` desde la pestaña Files and versions del dataset en Hugging Face (`bitext/Bitext-customer-support-llm-chatbot-training-dataset`).
2. Abrir colab.research.google.com y subir `Derivacion_Tickets_Colab.ipynb`.
3. Entorno de ejecución, Ejecutar todo. Cuando la celda de datos lo pida, subir el CSV.

Dependencias: pandas, scikit-learn, matplotlib y joblib, que ya vienen en Colab. Para ejecutarlo fuera de Colab, dejar el CSV en la misma carpeta del notebook e instalarlas con pip. No se necesita ninguna herramienta de pago.

## Datos

- Fuente: Bitext Customer Support Training Dataset, Hugging Face. 26.872 filas, en inglés.
- Licencia indicada en la ficha del dataset: cdla-sharing-1.0.
- Se usan dos columnas: `instruction` (texto del ticket) y `category` (11 categorías).
- Son textos generados, no tickets reales. No contienen datos personales.
- Hay 24.635 textos únicos. La partición (75 % entrenamiento, 25 % prueba) mantiene juntos los textos idénticos para evitar fuga.

## Arquitectura

```
ticket {"texto": ...}
  -> validación: los errores bloquean y el modelo no se ejecuta; las advertencias solo avisan
  -> modelo guardado: TF-IDF (palabras y pares de palabras) + regresión logística, sin fit()
  -> salida: categoría, confianza, decisión (umbral 0,6) y mensaje
```

Contrato de entrada: un diccionario con `texto`, de 5 a 500 caracteres. Advertencias: menos de 50 % de letras, o ninguna palabra conocida por el modelo. El 50 % es un criterio propio, sin calibrar.

## Resultados (ejecución real)

- Prueba de 6.729 tickets: exactitud 0,997 y F1 macro 0,998. Hubo 18 errores, 2 de ellos con confianza de 0,6 o más.
- Con umbral 0,6: se derivan solos 98,2 % de los tickets, con 100 % de aciertos entre ellos.
- Comparación con validación cruzada: Naive Bayes 0,996, SVM lineal 0,998, regresión logística 0,997, línea base 0,223.
- Con 16 tickets escritos a mano fuera de las plantillas: se derivan solos 10, todos correctos, y la sugerencia es correcta en 12 de 16.

## Limitaciones y riesgos

- Los tickets son generados con plantillas, más ordenados que los reales, y están en inglés: la exactitud de 0,997 es optimista.
- Las categorías vienen del dataset y no corresponden a las áreas de una institución concreta.
- Umbral y criterio de 50 % sin calibrar con datos reales.
- Sin línea base del proceso actual de derivación (tiempo y porcentaje correcto).
- Con tickets reales habría datos personales: deben anonimizarse y contar con autorización.
- El modelo guardado con joblib es un archivo pickle: cargar solo archivos de fuente confiable.
- El repositorio no contiene credenciales, claves ni archivos .env.

## Uso de IA generativa

Se usó Claude (Anthropic) para el código del notebook, este README, Los resultados provienen de ejecutar el notebook completo. La declaración detallada está en el ipynb
