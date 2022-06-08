<template>
  <div class="vue-modal" v-show="open">
      <transition name="fade">
        <div class="veu-modal-inner">
          <div class="vue-modal-content">
            <slot />
            <div class="right">
              <button class="btn btn__primary" @click="$emit('close')">X</button>
            </div>
            <h3>Resultado de la Validación del Esquema ({{schemaName}})</h3>
            <button v-if="resultData === 'The file corresponds to the schema!'" class="btn btn__success btn__lg">{{resultData}}</button>
            <button v-else class="btn btn__danger btn__lg">{{resultData}}</button>
            <hr />
            <h4>Data de Ejemplo</h4>
            <div class="containerTable">      
            <table id="firstTable">
              <thead>
                <tr>
                  <th v-for="(colum, index) in example.columns"
              :key="index">{{colum}}</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(row, index) in example.data" :key="index">
                  <td v-for="(field, index) in row" :key="index">{{field}}</td>
                </tr>
              </tbody>
            </table>
            </div>
            <br />
            <hr />
            <div v-if="resultData != 'The file corresponds to the schema!'">
            <h4>Errores Encontrados</h4>
            <div class="containerErrorTable">      
            <table id="secondTable">
              <thead>
                <tr>
                  <th>Fila</th>
                  <th>Columna</th>
                  <th>Valor</th>
                  <th>Descripcion</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(row, index) in errorData.data" :key="index">
                  <td v-for="(field, index) in row" :key="index">{{field}}</td>
                </tr>
              </tbody>
            </table>
            </div>
            </div>
            <hr />
            <br />
          </div>
        </div>
      </transition>
    </div>
</template>

<script>
import { onMounted, onUnmounted } from "vue";
  export default {
    name: 'ModalDirection',
    props: {
    open: {
      type: Boolean,
      required: true
    },
    example: Object,
    resultData: String,
    errorData: Object,
    schemaName: String
  },
  setup(_, { emit }) {
    const close = () => {
      emit("close");
    };

    const handleKeyup = (event) => {
      if (event.keyCode === 27) {
        close();
      }
    };

    onMounted(() => document.addEventListener("keyup", handleKeyup));
    onUnmounted(() => document.removeEventListener("keyup", handleKeyup));

    return { close,
    
    };
  },
  data() {
      return {
      
      }
  }

  };
</script>

<style>
.vue-modal {
  position: fixed;
  top: 0;
  left: 0.5;
  width: 100%;
  overflow-x: auto;
  overflow-y: scroll;
  background-color: rgba(0, 0, 0, 0.4);
  z-index: 1;
}

.vue-modal-inner {
  max-width: 500px;
  margin: 2rem auto;
}

.vue-modal-content {
  position:fixed;
  max-width: 75%;
  height: 100%;
  left: 10;
  background-color: #fff;
  border: 1px solid rgba(0, 0, 0, 0.3);
  background-clip: padding-box;
  border-radius: 0.3rem;
  padding: 1rem;
  overflow-y: scroll;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}

.fade-enter,
.fade-leave-to {
  opacity: 0;
}

table {
  font: 1.5rem/1.2 'Open Sans', sans-serif;
  font-family: 'Open Sans', sans-serif;
  width: 90%;
  border-collapse: collapse;
  border: 3px solid #44475C;
  margin: 5px 5px 0 5px;
}

table th {
  text-transform: uppercase;
  text-align: left;
  background: #44475C;
  color: #FFF;
  padding: 5px;
  min-width: 10px;
}

table td {
  text-align: left;
  padding: 3px;
  border-right: 2px solid #7D82A8;
}
table td:last-child {
  border-right: none;
}
table tbody tr:nth-child(2n) td {
  background: #D4D8F9;
}

.containerTable {
    width: 100%;
    overflow-x: auto;
    overflow-y: scroll;
    white-space: nowrap;
}

.containerErrorTable {
  width: 100%;
  overflow-x: auto;
  overflow-y: scroll;
}

.right {
  text-align: right;
}
</style>
