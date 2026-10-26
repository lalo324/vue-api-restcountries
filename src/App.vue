
<script setup>
import { ref, computed, onMounted } from 'vue'

const paises = ref([])
const cargando = ref(false)
const error = ref('')
const busqueda = ref('')

async function obtenerPaises() {
  cargando.value = true
  error.value = ''

  try {
    const respuesta = await fetch('https://countries.dev/countries')

    if (!respuesta.ok) {
      throw new Error(`Error HTTP: ${respuesta.status}`)
    }

    const resultado = await respuesta.json()

    // Admite respuestas en forma de lista u objeto.
    let lista = Array.isArray(resultado)
      ? resultado
      : Array.isArray(resultado.data)
        ? resultado.data
        : Array.isArray(resultado.countries)
          ? resultado.countries
          : resultado.data && typeof resultado.data === 'object'
            ? Object.values(resultado.data)
            : []

    if (lista.length === 0) {
      throw new Error('La API no devolvió una lista de países.')
    }

    paises.value = lista.map((pais) => ({
      nombre:
        pais.name?.common ||
        pais.name ||
        pais.commonName ||
        'Nombre no disponible',
      capital: Array.isArray(pais.capital)
        ? pais.capital[0] || 'Sin capital'
        : pais.capital || 'Sin capital',
      region: pais.region || pais.continent || 'Sin región',
      poblacion: pais.population ?? 0,
      bandera:
        pais.flags?.png ||
        pais.flags?.svg ||
        pais.flag ||
        '',
      codigo: (
        pais.cca2 ||
        pais.alpha2Code ||
        pais.alpha2 ||
        pais.code ||
        ''
      ).toLowerCase()
    }))

    paises.value.sort((a, b) =>
      a.nombre.localeCompare(b.nombre)
    )
  } catch (e) {
    console.error('Error al consultar países:', e)
    error.value =
      'No se pudieron cargar los países. Intenta nuevamente.'
  } finally {
    cargando.value = false
  }
}

const paisesFiltrados = computed(() => {
  const texto = busqueda.value.trim().toLowerCase()

  if (!texto) return paises.value

  return paises.value.filter((pais) =>
    pais.nombre.toLowerCase().includes(texto) ||
    pais.capital.toLowerCase().includes(texto) ||
    pais.region.toLowerCase().includes(texto)
  )
})

onMounted(obtenerPaises)
</script>

<template>
  <div class="pagina">
    <header class="encabezado">
      <div class="marca">
        <span class="globo">🌍</span>
        <span>WORLD<span class="marca-color">EXPLORER</span></span>
      </div>
      <span class="etiqueta">PROYECTO VUE 3 · UNIDAD 3</span>
    </header>

    <main class="contenido">
      <section class="presentacion">
        <span class="sobretitulo">DESCUBRE EL MUNDO</span>
        <h1>Explorador de <span>Países</span></h1>
        <p>
          Explora países, conoce sus capitales y descubre
          información de diferentes regiones del planeta.
        </p>

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
        <div class="campo-busqueda">
          <span>🔎</span>
          <input
            id="busqueda"
            v-model="busqueda"
            type="search"
            placeholder="Busca por país, capital o región..."
          />
        </div>
      </section>

      <section class="seccion-paises">
        <div class="titulo-lista">
          <div>
            <h2>Países del mundo</h2>
            <p>Información consultada desde countries.dev.</p>
          </div>
          <button
            class="boton-actualizar"
            @click="obtenerPaises"
            :disabled="cargando"
          >
            {{ cargando ? 'Cargando...' : '↻ Actualizar' }}
          </button>
        </div>

        <div v-if="cargando" class="mensaje">
          <span class="spinner"></span>
          <h3>Cargando países...</h3>
          <p>Estamos consultando la información.</p>
        </div>

        <div v-else-if="error" class="mensaje">
          <span class="icono-aviso">⚠️</span>
          <h3>No se pudieron cargar los países</h3>
          <p>{{ error }}</p>
          <button class="boton-principal" @click="obtenerPaises">
            Intentar de nuevo
          </button>
        </div>

        <div
          v-else-if="paisesFiltrados.length === 0"
          class="mensaje"
        >
          <span class="icono-aviso">🔎</span>
          <h3>No se encontraron países</h3>
          <p>Prueba con otro nombre, capital o región.</p>
        </div>

        <div v-else class="rejilla">
          <article
            v-for="(pais, indice) in paisesFiltrados"
            :key="pais.codigo || pais.nombre + indice"
            class="tarjeta"
          >
            <img
              v-if="pais.bandera"
              class="bandera"
              :src="pais.bandera"
              :alt="'Bandera de ' + pais.nombre"
              loading="lazy"
            />
            <div v-else class="bandera-sin-imagen">🌐</div>

            <div class="datos-pais">
              <h3>{{ pais.nombre }}</h3>
              <p><span>Capital:</span> {{ pais.capital }}</p>
              <p><span>Región:</span> {{ pais.region }}</p>
              <p>
                <span>Población:</span>
                {{ Number(pais.poblacion).toLocaleString('es-MX') }}
              </p>
            </div>
          </article>
        </div>
      </section>
    </main>

    <footer>
      <p>WORLD EXPLORER · Desarrollado con Vue 3 y Fetch API</p>
    </footer>
  </div>
</template>

<style>
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  background: #f0f2f7;
  font-family: Inter, "Segoe UI", Arial, sans-serif;
  color: #e5eaf5;
}

button,
input {
  font: inherit;
}

.pagina {
  width: min(100%, 1100px);
  min-height: 100vh;
  margin: 0 auto;
  background: #090f20;
  box-shadow: 0 0 35px #17223b25;
}

.encabezado {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 12px;
  padding: 25px 30px;
  border-bottom: 1px solid #27354d;
}

.marca {
  display: flex;
  align-items: center;
  gap: 10px;
  font-weight: 800;
  letter-spacing: 1px;
}

.globo {
  font-size: 26px;
}

.marca-color {
  color: #5ce0bf;
}

.etiqueta,
.sobretitulo {
  color: #a8b8d5;
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 1.4px;
}

.contenido {
  padding: 35px 30px 55px;
}

.presentacion {
  text-align: center;
  max-width: 650px;
  margin: 0 auto 40px;
}

.sobretitulo {
  color: #5ce0bf;
}

.presentacion h1 {
  margin: 14px 0;
  color: #f1f5ff;
  font-size: clamp(32px, 6vw, 52px);
  line-height: 1.15;
}

.presentacion h1 span {
  color: #5ce0bf;
}

.presentacion > p {
  color: #b9c8df;
  line-height: 1.8;
}

.estadisticas {
  display: flex;
  justify-content: center;
  gap: 65px;
  margin-top: 30px;
}

.estadisticas div {
  display: grid;
  gap: 8px;
}

.estadisticas strong {
  color: #5ce0bf;
  font-size: 28px;
  font-weight: 500;
}

.estadisticas span {
  color: #bdc9dd;
  font-size: 12px;
}

.buscador {
  margin-bottom: 38px;
}

.buscador label {
  display: block;
  margin-bottom: 12px;
  color: #bdc9dd;
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 1px;
}

.campo-busqueda {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 0 16px;
  background: #131e33;
  border: 1px solid #30415d;
  border-radius: 12px;
}

.campo-busqueda input {
  width: 100%;
  padding: 17px 0;
  color: white;
  background: transparent;
  border: none;
  outline: none;
}

.campo-busqueda input::placeholder {
  color: #8092b2;
}

.titulo-lista {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 15px;
  margin-bottom: 22px;
}

.titulo-lista h2 {
  margin: 0 0 8px;
  color: #f1f5ff;
  font-size: 23px;
  font-weight: 500;
}

.titulo-lista p,
.mensaje p {
  color: #aebed8;
  font-size: 13px;
  line-height: 1.6;
}

.boton-actualizar {
  flex-shrink: 0;
  padding: 12px 15px;
  color: #5ce0bf;
  background: #131e33;
  border: 1px solid #344765;
  border-radius: 9px;
  cursor: pointer;
}

.boton-actualizar:disabled {
  opacity: 0.6;
  cursor: wait;
}

.rejilla {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(230px, 1fr));
  gap: 18px;
}

.tarjeta {
  overflow: hidden;
  background: #131e33;
  border: 1px solid #293b57;
  border-radius: 13px;
  transition: transform 0.2s, border-color 0.2s;
}

.tarjeta:hover {
  transform: translateY(-3px);
  border-color: #5ce0bf;
}

.bandera {
  display: block;
  width: 100%;
  height: 145px;
  object-fit: cover;
  background: #202d44;
}

.bandera-sin-imagen {
  display: grid;
  height: 145px;
  place-items: center;
  font-size: 42px;
  background: #202d44;
}

.datos-pais {
  padding: 16px;
}

.datos-pais h3 {
  margin: 0 0 14px;
  color: #5ce0bf;
  font-size: 18px;
}

.datos-pais p {
  margin: 8px 0;
  color: #d2dced;
  font-size: 13px;
  overflow-wrap: anywhere;
}

.datos-pais p span {
  color: #91a6c8;
}

.mensaje {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 245px;
  padding: 25px;
  text-align: center;
  background: #131e33;
  border: 1px solid #293b57;
  border-radius: 14px;
}

.mensaje h3 {
  margin: 14px 0 4px;
  color: #e5eaf5;
}

.icono-aviso {
  font-size: 30px;
}

.boton-principal {
  margin-top: 10px;
  padding: 13px 20px;
  color: #081521;
  background: #5ce0bf;
  border: none;
  border-radius: 9px;
  font-weight: 700;
  cursor: pointer;
}

.spinner {
  width: 35px;
  height: 35px;
  border: 3px solid #30415d;
  border-top-color: #5ce0bf;
  border-radius: 50%;
  animation: girar 0.8s linear infinite;
}

@keyframes girar {
  to {
    transform: rotate(360deg);
  }
}

footer {
  padding: 22px;
  text-align: center;
  border-top: 1px solid #27354d;
  color: #8395b5;
  font-size: 12px;
}

@media (max-width: 600px) {
  .encabezado {
    align-items: flex-start;
    flex-direction: column;
    padding: 20px;
  }

  .contenido {
    padding: 30px 18px;
  }

  .titulo-lista {
    align-items: flex-start;
    flex-direction: column;
  }

  .rejilla {
    grid-template-columns: 1fr;
  }
}
</style>
