
<script setup>
import { ref, computed, onMounted } from 'vue'

const paises = ref([])
const busqueda = ref('')
const cargando = ref(true)
const error = ref('')


async function obtenerPaises() {
  cargando.value = true
  error.value = ''

  try {
    const todosLosPaises = []
    let offset = 0
    let hayMas = true

    while (hayMas) {
      const respuesta = await fetch(
        `https://api.restcountries.com/countries/v5?limit=100&offset=${offset}&pretty=1`,
        {
          headers: {
            Authorization: `Bearer ${import.meta.env.VITE_REST_COUNTRIES_API_KEY}`
          }
        }
      )

      if (!respuesta.ok) {
        throw new Error(`Error HTTP: ${respuesta.status}`)
      }

      const resultado = await respuesta.json()
      const datos = resultado.data?.objects

      let listaPaises = []

      if (Array.isArray(datos)) {
        listaPaises = datos
      } else if (datos && typeof datos === 'object') {
        listaPaises = Object.values(datos)
      }

      todosLosPaises.push(...listaPaises)
      hayMas = resultado.data?.meta?.more === true
      offset += 100
    }

    paises.value = todosLosPaises

    if (paises.value.length === 0) {
      error.value = 'La API respondió, pero no devolvió una lista de países.'
    }
  } catch (e) {
    console.error('Error al consultar REST Countries:', e)
    error.value = `No se pudieron cargar los países: ${e.message}`
  } finally {
    cargando.value = false
  }
} 


function nombrePais(pais) {
  const nombre = pais.names ?? pais.name

  if (typeof nombre === 'string') return nombre

  return nombre?.common ??
    nombre?.name ??
    nombre?.official ??
    pais.common ??
    'Nombre no disponible'
}




function banderaPais(pais) {
  const codigo = pais.codes?.alpha_2

  if (!codigo) return ''

  return `https://flagcdn.com/w320/${codigo.toLowerCase()}.png`
}




function capitalPais(pais) {
  const capital = pais.capital ?? pais.capitals

  if (Array.isArray(capital)) {
    return capital.map(item =>
      typeof item === 'string' ? item : item?.name ?? ''
    ).filter(Boolean).join(', ') || 'Sin capital'
  }

  if (typeof capital === 'string') return capital

  return capital?.name ?? 'Sin capital'
}

function regionPais(pais) {
  return pais.region?.name ??
    pais.region ??
    pais.continents?.[0] ??
    'No disponible'
}

function poblacionPais(pais) {
  const valor = pais.population?.total ??
    pais.population?.value ??
    pais.population

  return typeof valor === 'number'
    ? valor.toLocaleString('es-MX')
    : 'No disponible'
}

const paisesFiltrados = computed(() => {
  const texto = busqueda.value.trim().toLowerCase()

  if (!texto) return paises.value

  return paises.value.filter(pais =>
    nombrePais(pais).toLowerCase().includes(texto) ||
    String(regionPais(pais)).toLowerCase().includes(texto) ||
    capitalPais(pais).toLowerCase().includes(texto)
  )
})

onMounted(obtenerPaises)
</script>

<template>
  <div class="pagina">
    <header class="encabezado">
      <div class="marca">
        <span class="globo">🌎</span>
        <span>WORLD<span class="marca-color">EXPLORER</span></span>
      </div>
      <span class="etiqueta">PROYECTO VUE 3 · UNIDAD 3</span>
    </header>

    <main class="contenido">
      <section class="presentacion">
        <span class="sobretitulo">DESCUBRE EL MUNDO</span>
        <h1>Explorador de <span>Países</span></h1>
        <p>Explora países, conoce sus capitales y descubre información de diferentes regiones del planeta.</p>

        <div class="estadisticas">
          <div>
            <strong>{{ paises.length }}</strong>
            <span>Países cargados</span>
          </div>
          <div>
            <strong>{{ paisesFiltrados.length }}</strong>
            <span>Resultados</span>
          </div>
        </div>
      </section>

      <section class="buscador">
        <label for="busqueda">BUSCA TU PRÓXIMO DESTINO</label>
        <div class="campo">
          <span>🔎</span>
          <input
            id="busqueda"
            v-model="busqueda"
            type="search"
            placeholder="Busca por país, capital o región..."
          />
          <button v-if="busqueda" class="limpiar" @click="busqueda = ''">✕</button>
        </div>
      </section>

      <section class="resultados">
        <div class="titulo-resultados">
          <div>
            <h2>Países del mundo</h2>
            <p>Información consultada desde REST Countries.</p>
          </div>
          <button class="boton-recargar" @click="obtenerPaises" :disabled="cargando">
            ↻ Actualizar
          </button>
        </div>

        <div v-if="cargando" class="estado">
          <div class="spinner"></div>
          <h3>Cargando países...</h3>
          <p>Estamos consultando la información.</p>
        </div>

        <div v-else-if="error" class="estado">
          <span class="icono-estado">⚠️</span>
          <h3>No se pudieron cargar los países</h3>
          <p>{{ error }}</p>
          <button class="boton" @click="obtenerPaises">Intentar de nuevo</button>
        </div>

        <div v-else-if="paisesFiltrados.length === 0" class="estado">
          <span class="icono-estado">🔍</span>
          <h3>No encontramos países</h3>
          <p>Prueba con otro nombre, capital o región.</p>
        </div>

        <div v-else class="rejilla">
          <article
            v-for="(pais, indice) in paisesFiltrados"
            :key="pais.cca3 ?? pais.code ?? nombrePais(pais) + indice"
            class="tarjeta"
          >
            <div class="bandera">
              <img v-if="banderaPais(pais)" :src="banderaPais(pais)" :alt="'Bandera de ' + nombrePais(pais)" />
              <span v-else class="bandera-faltante">🌐</span>
            </div>
            <div class="datos">
              <span class="region">{{ regionPais(pais) }}</span>
              <h3>{{ nombrePais(pais) }}</h3>

              <div class="detalle">
                <span>🏙️</span>
                <span><small>Capital</small><strong>{{ capitalPais(pais) }}</strong></span>
              </div>
              <div class="detalle">
                <span>🌍</span>
                <span><small>Región</small><strong>{{ regionPais(pais) }}</strong></span>
              </div>
              <div class="detalle">
                <span>👥</span>
                <span><small>Población</small><strong>{{ poblacionPais(pais) }}</strong></span>
              </div>
            </div>
          </article>
        </div>
      </section>
    </main>

    <footer>
      <span>🌐 WORLD EXPLORER</span>
      <span>Desarrollado con Vue 3 · fetch() · REST Countries</span>
    </footer>
  </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=DM+Sans:wght@400;500;600;700&family=Manrope:wght@400;500;600;700;800&display=swap');

:root {
  font-family: 'DM Sans', sans-serif;
  color: #e8edf8;
  background: #0a1020;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
}

* { box-sizing: border-box; }

body {
  margin: 0;
  min-width: 320px;
  min-height: 100vh;
}

button, input { font: inherit; }

.pagina {
  min-height: 100vh;
  background: radial-gradient(ellipse at 50% 0%, #17294a 0%, transparent 45%), #0a1020;
}

.encabezado {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 16px;
  padding: 24px 7%;
  border-bottom: 1px solid #263149;
}

.marca {
  display: flex;
  align-items: center;
  gap: 10px;
  font-family: 'Manrope', sans-serif;
  font-weight: 800;
  letter-spacing: 1px;
}

.globo { font-size: 27px; }
.marca-color, .presentacion h1 span { color: #61dfbd; }

.etiqueta, .sobretitulo {
  color: #75e4c5;
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 2px;
}

.contenido { width: min(1180px, 88%); margin: auto; }

.presentacion { padding: 68px 0 38px; text-align: center; }

.presentacion h1 {
  margin: 15px 0;
  font-family: 'Manrope', sans-serif;
  font-size: clamp(36px, 6vw, 60px);
  font-weight: 800;
  letter-spacing: -2px;
}

.presentacion > p {
  max-width: 570px;
  margin: auto;
  color: #aab5cb;
  line-height: 1.8;
}

.estadisticas {
  display: flex;
  justify-content: center;
  gap: 55px;
  margin-top: 32px;
}

.estadisticas div { display: flex; flex-direction: column; gap: 5px; }
.estadisticas strong { font-family: 'Manrope', sans-serif; font-size: 25px; color: #61dfbd; }
.estadisticas span { color: #aab5cb; font-size: 12px; }

.buscador { max-width: 740px; margin: 0 auto 55px; }

.buscador label {
  display: block;
  margin-bottom: 12px;
  color: #aab5cb;
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 1.5px;
}

.campo {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 7px 17px;
  background: #131d31;
  border: 1px solid #35435d;
  border-radius: 12px;
}

.campo:focus-within { border-color: #61dfbd; box-shadow: 0 0 0 3px #61dfbd18; }
.campo input { width: 100%; padding: 12px 0; color: white; background: transparent; border: none; outline: none; }
.campo input::placeholder { color: #75829a; }
.limpiar { color: #aab5cb; background: transparent; border: 0; cursor: pointer; }

.titulo-resultados {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 15px;
  margin-bottom: 22px;
}

.titulo-resultados h2 { margin: 0 0 7px; font-family: 'Manrope', sans-serif; font-size: 25px; }
.titulo-resultados p { margin: 0; color: #94a1b9; font-size: 13px; }

.boton-recargar {
  padding: 10px 14px;
  color: #61dfbd;
  background: #131d31;
  border: 1px solid #35435d;
  border-radius: 8px;
  cursor: pointer;
}

.boton-recargar:disabled { opacity: 0.5; cursor: wait; }

.rejilla {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 22px;
  padding-bottom: 60px;
}

.tarjeta { overflow: hidden; background: #121c30; border: 1px solid #293750; border-radius: 15px; transition: transform .2s, border-color .2s; }
.tarjeta:hover { transform: translateY(-4px); border-color: #61dfbd80; }

.bandera {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 175px;
  overflow: hidden;
  background: #202c42;
}

.bandera img { display: block; width: 100%; height: 100%; object-fit: cover; }
.bandera-faltante { font-size: 42px; }
.datos { padding: 20px; }

.region { color: #61dfbd; font-size: 10px; font-weight: 700; letter-spacing: 1.5px; text-transform: uppercase; }
.datos h3 { margin: 9px 0 22px; font-family: 'Manrope', sans-serif; font-size: 20px; }

.detalle {
  display: flex;
  align-items: flex-start;
  gap: 11px;
  padding: 11px 0;
  border-top: 1px solid #293750;
}

.detalle > span:first-child { font-size: 17px; }
.detalle > span:last-child { display: flex; flex-direction: column; gap: 4px; min-width: 0; }
.detalle small { color: #8f9cb4; font-size: 11px; }
.detalle strong { overflow-wrap: anywhere; font-size: 13px; font-weight: 500; }

.estado {
  padding: 55px 20px;
  margin-bottom: 60px;
  text-align: center;
  background: #121c30;
  border: 1px solid #293750;
  border-radius: 15px;
}

.estado h3 { margin-bottom: 8px; }
.estado p { color: #aab5cb; line-height: 1.6; }

.spinner {
  width: 38px;
  height: 38px;
  margin: 0 auto 18px;
  border: 3px solid #34435c;
  border-top-color: #61dfbd;
  border-radius: 50%;
  animation: girar .8s linear infinite;
}

@keyframes girar { to { transform: rotate(360deg); } }
.icono-estado { font-size: 35px; }

.boton {
  padding: 12px 20px;
  color: #071a18;
  font-weight: 700;
  background: #61dfbd;
  border: none;
  border-radius: 8px;
  cursor: pointer;
}

footer {
  display: flex;
  justify-content: space-between;
  gap: 15px;
  padding: 25px 7%;
  color: #8491a9;
  border-top: 1px solid #263149;
  font-size: 11px;
}

footer span:first-child { color: #61dfbd; font-weight: 700; letter-spacing: 1px; }

@media (max-width: 850px) {
  .rejilla { grid-template-columns: repeat(2, minmax(0, 1fr)); }
}

@media (max-width: 550px) {
  .encabezado { padding: 18px 5%; }
  .etiqueta { max-width: 120px; font-size: 9px; text-align: right; }
  .contenido { width: 90%; }
  .presentacion { padding-top: 48px; }
  .estadisticas { gap: 30px; }
  .rejilla { grid-template-columns: 1fr; }
  .bandera { height: 190px; }
  .titulo-resultados { align-items: flex-start; }
  footer { flex-direction: column; align-items: center; text-align: center; }
}
</style>
