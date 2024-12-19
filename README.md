# Limpieza de datos Plaza Vea
Se me encargó reducir el número de categorías, subcategorías y grupos de una base de datos con productos de Plaza Vea. **La base de datos no fue construida por mí, por lo que no la adjuntaré en este repositorio. Sin embargo, comentaré el proceso que seguí para cumplir con el encargo.** \
\
La base original consistía en alrededor de **8 millones de entradas de productos de Plaza Vea**, con variables como el nombre del producto, su categoría, subcategoría, grupo, descripción, precio de lista, precio online, y cuatro variables dummy para señalar si contaban con octógonos de grasas saturadas, azúcar y sodio. **Eran 182 subcategorías y 21 categorías. El resultado final fue una base con 78 subcategorías y 9 categorías, eliminando solo dos observaciones redundantes.**\
\
![image](https://github.com/RodrigoCandelaApaza/Limpieza-de-datos-Plaza-Vea/assets/58021217/4d681518-5e1f-4786-a078-2e28a2dd5de3) \
\
Para lograrlo, fue necesario identificar las subcategorías que deberían estar juntas. Por ejemplo, la categoría `abarrotes` contenía 22 subcategorías, entre las que se encontraban `salsas_cremas_condimentos`, `condimentos` y `cremas_salsas`. Junté estas tres subcategorías en una sola llamada `salsas_condimentos`. Además, existía un problema con los nombres de las categorías y subcategorías, los cuales eran muy parecidos entre sí con excepción de algunos caracteres. Para solucionarlo reemplacé los nombres de todas las observaciones para eliminar estos caracteres y reemplazarlos por "_":
````python
# Elimino espacios y conectores en todas las subcategorias
for i in ['/',' y ',', ', '  ', ' ']:
  b3['subcategoria'] = b3['subcategoria'].str.replace(i,'_')
````
Después de repetir este proceso para otras ocurrencias, conseguí reducir las subcategorías de 22 a 12 dentro de `abarrotes`. Repetí el mismo proceso para las categorías `hogar_limpieza`, `cuidado_bebe`, `cuidado_personal`, `frutas_verduras`, `carnes_aves_pescados`, `lacteos_quesos_fiambres_huevos`, `panaderia_pasteleria`, `bebidas`, `mascotas`, `frescos` y `na`. \
\
Sin embargo, existían productos que no estaban asignados a una subcategoría adecuada. Afortunadamente, las variables `descripción`e `id` contenían más información sobre el producto. Por ejemplo, la categoría `mascotas` contenía varias subcategorías, entre las que estaban `accesorios`, `accesorios_gatos` y `accesorios_perros`. La subcategoría `accesorios` contenía también accesorios para gatos y perros, por lo que revisé cada observación dentro de ella y moví a `accesorios_gatos` aquellas que contenían las palabras "gato" o "arena". Estos procedimientos no eran menores; en ocasiones, la cantidad de productos mal categorizados era considerable. Por ejemplo, 486 productos de la subcategoría `accesorios` contenían en su descripción la palabra "gato", por lo que fueron movidas a `accesorios_gatos`. Este proceso lo realicé para todas las subcategorías demasiado generales, con ayuda de las funciones `buscar_palabra()` y `separar_subcategorias()`:

**Función buscar_palabra()**
````python
def buscar_palabra(cat, subcat_actual, col_busqueda, palabra, subcat_nueva):
  """
      Busca una palabra en col_busqueda, y cambia las subcategorias de las observaciones que la contengan a una subcategoría nueva.
      Argumentos:
      - cat (str): La categoría donde se está trabajando.
      - subcat_actual (str): La subcategoría donde se está trabajando.
      - col_busqueda (str): La columna donde se quiere buscar la palabra clave.
      - palabra (list or str): La(s) palabra(s) que se quiere buscar.
      - subcat_nueva (str): La subcategoría a donde se dirigirán las observaciones que contengan la palabra clave.
  """
  if isinstance(palabra,list): # si se pasa una lista de palabras como argumento
    total = 0
    for p in palabra:
      # Cuenta el número de veces que se encontró la palabra
      veces = b3.loc[(b3['categoria']==cat)&(b3['subcategoria']==subcat_actual)&(b3[col_busqueda].str.contains(p,regex=False)), col_busqueda].count()
      # Asigna las observaciones coincidentes a la subcategoría deseada.
      print('Buscando la palabra {}...'.format(p))
      b3.loc[(b3['categoria']==cat)&(b3['subcategoria']==subcat_actual)&(b3[col_busqueda].str.contains(p,regex=False)), 'subcategoria'] = subcat_nueva
      total += veces
  elif isinstance(palabra, str): # si se pasa una sola palabra como argumento
      # Cuenta el número de veces que se encontró la palabra
      total = b3.loc[(b3['categoria']==cat)&(b3['subcategoria']==subcat_actual)&(b3[col_busqueda].str.contains(palabra,regex=False)), col_busqueda].count()
      # Asigna las observaciones coincidentes a la subcategoría deseada.
      print('Buscando la palabra {}...'.format(palabra))
      b3.loc[(b3['categoria']==cat)&(b3['subcategoria']==subcat_actual)&(b3[col_busqueda].str.contains(palabra,regex=False)), 'subcategoria'] = subcat_nueva
  else:
    return 'El argumento "palabra" debe ser de formato list o string.'
  # Imprime el resultado
  print('Se movieron {} elementos a la subcategoria {}.'.format(total,subcat_nueva))
````
\
**Función separar_subcategorias()**
````python
def separar_subcategorias(cat,subcat_actual,grupos_subcat_nuevas):
  """
      Separa una subcategoría que agrega muchas otras subcategorias.
      Argumentos:
      - cat (str): La categoría del producto.
      - subcat_actual (str): La subcategoría actual del producto, la cual comprende otras subcategorias.
      - grupos_subcat_nuevas (dict): Diccionario de strings donde las llaves son el grupo al que pertenece el producto y
        los valores la subcategoría a la que realmente pertenece.
  """
  # Contar cuántos valores hay inicialmente en la subcategoría
  counts_iniciales = len(b3.loc[(b3['categoria']==cat) & (b3['subcategoria']==subcat_actual),'subcategoria'])
  # Hace la reasignación de subcategorías
  for grupo, subcat_nueva in grupos_subcat_nuevas.items():
    b3.loc[(b3['categoria']==cat) & (b3['subcategoria']==subcat_actual) & (b3['grupo']==grupo),'subcategoria'] = subcat_nueva
  # Contar cuántos valores hay tras el proceso en la subcategoría
  counts_finales = len(b3.loc[(b3['categoria']==cat) & (b3['subcategoria']==subcat_actual),'subcategoria'])

  print('{} valores correctamente reasignados. Hay {} valores restantes en la subcategoria {}.'.format(counts_iniciales-counts_finales, counts_finales, subcat_actual))
````
Este proyecto de limpieza de datos mejoró mis habilidades en manipulación de datos masivos, normalización de categorías y automatización en Python, fortaleciendo mi experiencia práctica en gestión y limpieza de datos.
