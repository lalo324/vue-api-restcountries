
<script setup>
import { ref, computed, onMounted } from 'vue'

const paises = ref([])
const cargando = ref(true)
const error = ref('')
const busqueda = ref('')

// Convierte los códigos internacionales en nombres de países en español.
const nombresPaises = new Intl.DisplayNames(['es'], {
  type: 'region'
})

function obtenerNombreEspanol(pais) {
  const codigo = pais.cca2 || pais.alpha2Code || pais.alpha2

  if (!codigo) {
    return pais.translations?.spa?.common ||
      pais.translations?.spa?.official ||
      pais.name?.common ||
      pais.name ||
      'País sin nombre'
  }

  try {
    return nombresPaises.of(codigo.toUpperCase()) || pais.name?.common || pais.name
  } catch {
    return pais.name?.common || pais.name || 'País sin nombre'
  }
}

function obtenerNombreOriginal(pais) {
  return pais.name?.common || pais.name || ''
}

function obtenerBandera(pais) {
  return pais.flags?.svg ||
    pais.flags?.png ||
    pais.flag ||
    ''
}

function obtenerCapital(pais) {
  const capital = pais.capital
  if (Array.isArray(capital)) return capital.join(', ') || 'No disponible'
  return capital || 'No disponible'
}

function obtenerRegion(pais) {
  const region = pais.region || pais.continent || ''
  const traducciones = {
    Africa: 'África',
    Americas: 'América',
    Asia: 'Asia',
    Europe: 'Europa',
    Oceania: 'Oceanía',
    Antarctic: 'Antártida',
    'North America': 'América del Norte',
    'South America': 'América del Sur'
  }
  return traducciones[region] || region || 'No disponible'
}

function obtenerPoblacion(pais) {
  const poblacion = Number(pais.population)
  return Number.isFinite(poblacion)
    ? poblacion.toLocaleString('es-MX')
    : 'No disponible'
}
  const paisesFiltrados = computed(() => {
  const texto = busqueda.value.trim().toLocaleLowerCase('es')

  return paises.value.filter(pais => {
    const nombreEspanol = obtenerNombreEspanol(pais)
      .toLocaleLowerCase('es')
    const nombreOriginal = obtenerNombreOriginal(pais)
      .toLocaleLowerCase('es')
    const capital = obtenerCapital(pais)
      .toLocaleLowerCase('es')

    return nombreEspanol.includes(texto) ||
      nombreOriginal.includes(texto) ||
      capital.includes(texto)
  })
})
onMounted(async () => {
  try {
    cargando.value = true
    error.value = ''

    const respuesta = await fetch('https://countries.dev/countries')

    if (!respuesta.ok) {
      throw new Error('No se pudieron obtener los países')
    }

    const datos = await respuesta.json()

    // La API puede devolver una lista o un objeto con la lista.
    const lista = Array.isArray(datos)
      ? datos
      : datos.countries || datos.data || []

    if (!Array.isArray(lista)) {
      throw new Error('El formato de los datos no es válido')
    }

    paises.value = lista
  } catch (e) {
    console.error(e)
    error.value = 'No pudimos cargar los países. Comprueba tu conexión e inténtalo de nuevo.'
  } finally {
    cargando.value = false
  }
})
</script>

<template>
  <div class="pagina">
    <header class="encabezado">
      <div class="marca">
        <span class="icono-mundo">🌎</span>
        <span>WorldExplorer</span>
      </div>
      <p>Descubre el mundo, país por país</p>
    </header>

    <main class="contenido">
      <section class="presentacion">
        <span class="etiqueta">EXPLORA EL PLANETA</span>
        <h1>El mundo en tus manos</h1>
        <p>
          Descubre países, conoce sus banderas y explora información
          de todo el mundo.
        </p>

        <div class="buscador">
          <span aria-hidden="true">🔎</span>
          <input
            v-model="busqueda"
            type="search"
            placeholder="Buscar un país, por ejemplo España..."
            aria-label="Buscar país"
          />
          <button
            v-if="busqueda"
            type="button"
            class="limpiar"
            @click="busqueda = ''"
          >
            Limpiar
          </button>
        </div>

        <p class="contador" v-if="!cargando && !error">
          {{ paisesFiltrados.length }} países encontrados
        </p>
      </section>

      <section class="seccion-paises">
        <div class="titulo-seccion">
          <div>
            <h2>Explora los países</h2>
            <p>Información para conocer mejor nuestro planeta.</p>
          </div>
        </div>

        <div v-if="cargando" class="estado">
          <div class="spinner"></div>
          <p>Cargando países...</p>
        </div>

        <div v-else-if="error" class="estado error">
          <span>⚠️</span>
          <p>{{ error }}</p>
          <button @click="window.location.reload()">Volver a intentar</button>
        </div>

        <div v-else-if="paisesFiltrados.length === 0" class="estado">
          <span>🔎</span>
          <p>No encontramos países que coincidan con tu búsqueda.</p>
          <button @click="busqueda = ''">Mostrar todos los países</button>
        </div>

        <div v-else class="rejilla">
          <article
            v-for="(pais, indice) in paisesFiltrados"
            :key="pais.cca3 || pais.cca2 || obtenerNombreOriginal(pais) || indice"
            class="tarjeta"
          >
            <div class="bandera-contenedor">
              <img
                v-if="obtenerBandera(pais)"
                :src="obtenerBandera(pais)"
                :alt="'Bandera de ' + obtenerNombreEspanol(pais)"
                class="bandera"
                loading="lazy"
              />
              <span v-else class="bandera-faltante">🌎</span>
            </div>

            <div class="datos-pais">
              <h3>{{ obtenerNombreEspanol(pais) }}</h3>

              <div class="dato">
                <span class="dato-icono">📍</span>
                <span>
                  <strong>Capital</strong>
                  {{ obtenerCapital(pais) }}
                </span>
              </div>

              <div class="dato">
                <span class="dato-icono">🗺️</span>
                <span>
                  <strong>Región</strong>
                  {{ obtenerRegion(pais) }}
                </span>
              </div>

              <div class="dato">
                <span class="dato-icono">👥</span>
                <span>
                  <strong>Población</strong>
                  {{ obtenerPoblacion(pais) }}
                </span>
              </div>
            </div>
          </article>
        </div>
      </section>
    </main>

    <footer class="pie">
      <p>WorldExplorer · Proyecto desarrollado con Vue 3 y una API REST</p>
    </footer>
  </div>
</template>

<style>
*,
*::before,
*::after {
  box-sizing: border-box;
}

:root {
  color-scheme: dark;
  font-family: Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont,
    "Segoe UI", sans-serif;
  background: #090f20;
  color: #f3f6ff;
}

html,
body,
#app {
  width: 100%;
  min-width: 320px;
  min-height: 100%;
  margin: 0;
}

body {
  background: #090f20;
}

button,
input {
  font: inherit;
}

button {
  cursor: pointer;
}

.pagina {
  min-height: 100vh;
  width: 100%;
  overflow: hidden;
  background:
    radial-gradient(ellipse at 50% 0%, rgba(37, 83, 128, 0.18), transparent 42%),
    #090f20;
}

.encabezado {
  width: 100%;
  padding: 22px 6%;
  border-bottom: 1px solid rgba(255, 255, 255, 0.09);
  background: rgba(11, 18, 36, 0.85);
}

.marca {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 21px;
  font-weight: 750;
  letter-spacing: -0.5px;
}

.icono-mundo {
  font-size: 27px;
}

.encabezado > p {
  margin: 5px 0 0 38px;
  color: #9ba9c4;
  font-size: 13px;
}

.contenido {
  width: 100%;
  margin: 0 auto;
  padding: 0 6%;
}

.presentacion {
  max-width: 850px;
  margin: 0 auto;
  padding: 64px 0 42px;
  text-align: center;
}

.etiqueta {
  display: inline-block;
  padding: 7px 13px;
  border: 1px solid rgba(92, 224, 191, 0.3);
  border-radius: 30px;
  background: rgba(92, 224, 191, 0.08);
  color: #5ce0bf;
  font-size: 11px;
  font-weight: 750;
  letter-spacing: 1.7px;
}

.presentacion h1 {
  margin: 20px 0 12px;
  font-size: clamp(32px, 5vw, 53px);
  line-height: 1.12;
  letter-spacing: -1.8px;
}

.presentacion > p:not(.contador) {
  max-width: 620px;
  margin: 0 auto;
  color: #a8b4cc;
  font-size: 15px;
  line-height: 1.7;
}

.buscador {
  display: flex;
  align-items: center;
  gap: 12px;
  width: 100%;
  max-width: 620px;
  min-height: 58px;
  margin: 30px auto 0;
  padding: 0 18px;
  border: 1px solid #293650;
  border-radius: 15px;
  background: #111a2e;
  box-shadow: 0 12px 38px rgba(0, 0, 0, 0.18);
}

.buscador > span {
  font-size: 18px;
}

.buscador input {
  flex: 1;
  width: 0;
  border: 0;
  outline: 0;
  background: transparent;
  color: #f3f6ff;
  font-size: 14px;
}

.buscador input::placeholder {
  color: #7886a1;
}

.limpiar {
  border: 0;
  background: transparent;
  color: #5ce0bf;
  font-size: 12px;
}

.contador {
  margin-top: 14px !important;
  color: #8190ad !important;
  font-size: 12px !important;
}

.seccion-paises {
  width: 100%;
  padding: 10px 0 60px;
}

.titulo-seccion {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 24px;
}

.titulo-seccion h2 {
  margin: 0 0 6px;
  font-size: 25px;
  letter-spacing: -0.5px;
}

.titulo-seccion p {
  margin: 0;
  color: #8997b2;
  font-size: 13px;
}

.rejilla {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(245px, 1fr));
  gap: 20px;
  width: 100%;
}

.tarjeta {
  min-width: 0;
  overflow: hidden;
  border: 1px solid #26324a;
  border-radius: 16px;
  background: linear-gradient(145deg, #121d32, #0e1729);
  transition: transform 0.2s ease, border-color 0.2s ease;
}

.tarjeta:hover {
  transform: translateY(-4px);
  border-color: #3e7b82;
}

.bandera-contenedor {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 150px;
  overflow: hidden;
  background: #17233a;
}

.bandera {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.bandera-faltante {
  font-size: 42px;
}

.datos-pais {
  padding: 18px;
}

.datos-pais h3 {
  margin: 0 0 17px;
  font-size: 19px;
  line-height: 1.3;
}

.dato {
  display: flex;
  align-items: flex-start;
  gap: 10px;
  margin-top: 13px;
  color: #d4dced;
  font-size: 12px;
  line-height: 1.5;
  overflow-wrap: anywhere;
}

.dato-icono {
  flex-shrink: 0;
  width: 18px;
}

.dato strong {
  display: block;
  margin-bottom: 2px;
  color: #8292b0;
  font-size: 10px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.6px;
}

.estado {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 12px;
  min-height: 220px;
  color: #a8b4cc;
  text-align: center;
}

.estado > span {
  font-size: 32px;
}

.estado button {
  padding: 10px 16px;
  border: 1px solid #36526b;
  border-radius: 9px;
  background: #14263a;
  color: #5ce0bf;
}

.spinner {
  width: 36px;
  height: 36px;
  border: 3px solid #293650;
  border-top-color: #5ce0bf;
  border-radius: 50%;
  animation: girar 0.8s linear infinite;
}

@keyframes girar {
  to {
    transform: rotate(360deg);
  }
}

.error {
  color: #ffb6b6;
}

.pie {
  padding: 24px 6%;
  border-top: 1px solid rgba(255, 255, 255, 0.08);
  color: #7886a1;
  font-size: 12px;
  text-align: center;
}

@media (max-width: 600px) {
  .encabezado {
    padding: 18px 5%;
  }

  .contenido {
    padding: 0 5%;
  }

  .presentacion {
    padding: 45px 0 32px;
  }

  .presentacion h1 {
    letter-spacing: -1px;
  }

  .rejilla {
    grid-template-columns: 1fr;
  }

  .titulo-seccion h2 {
    font-size: 22px;
  }
}
</style>
