<template>
    <h2>Formulario de Validación</h2>
    <hr />
    <div>
        <label><h4>Seleccione el Tipo de Validación:</h4></label>
        <label><input type="radio" v-model="showFirst" value="true" /><span></span>Schema de Artifactory</label><br />
        <label><input type="radio" v-model="showFirst" value="false" />Cargar Schema Personalizado</label>
    </div>
    <hr />
    <div v-if="showFirst === 'true'">
        <label><h4>Seleccione UUAA:</h4></label>
        <div class="dropdown">
          <select v-model="selectedUUAA" @change="onChangeUUAA">
              <option value="">----------</option>
              <option
              v-for="(uuaa, index) in listUUAA.data"
              :value="uuaa"
              :key="index"
              >
              {{ uuaa }}
              </option>
          </select>
      </div>
      <hr />
      <label><h4>Seleccione Tabla:</h4></label>
      <div class="dropdown">
      <select
        :disabled="listUUAA.length == 0"
        v-model="selectedTable"
      >
        <option value="">----------</option>
        <option
          v-for="(table, index) in listTable.tables"
          :key="index"
          :value="table"
        >
          {{ table }}
        </option>
      </select>
    </div>
     <hr />
     <label><h4>Seleccione Ambiente:</h4></label>
        <div class="dropdown">
          <select v-model="selectedEnvi" @change="onChangeEnvi">
              <option value="raw">RAW</option>
              <option value="master">MASTER</option>
          </select>
      </div>
      <hr />
      <label><h4>Cargue el Archivo a Validar:</h4></label>
    <div>
      <input
      type="file"
      style="display: block"
      ref="fileInput"
      accept="*"
      @change="handleFileUpload( $event )"/>
    </div>
    <hr />
    <br />
    <button class="btn btn__primary btn__lg" v-on:click="submitArtifactory()">Validar</button>
    </div>
  <div v-else>
    <label><h4>Cargue el Archivo de Datos a Validar:</h4></label>
    <div>
      <input
      type="file"
      style="display: block"
      ref="fileData"
      accept="*"
      @change="handleDataUpload( $event )"/>
    </div>
    <hr />
    <label><h4>Cargue el Archivo de Esquema a Validar:</h4></label>
    <div>
      <input
      type="file"
      style="display: block"
      ref="fileSchema"
      accept="*"
      @change="handleSchemaUpload( $event )"/>
    </div>
    <hr />
    <br />
    <button class="btn btn__primary btn__lg" v-on:click="submitWhitSchema()">Validar</button>
  </div>


  <ModalDirection 
    :open="isOpen" 
    @close="isOpen = !isOpen" 
    :example="exampleData" 
    :resultData="resultData" 
    :errorData="errorData"
    :schemaName="schemaName"
  >
   
  </ModalDirection>

</template>

<script>
import axios from "axios";
import ModalDirection from "@/components/Modal";
import { ref } from "vue";

export default {
  name: 'FormValidate',
  components: {
    ModalDirection
  },
  props: {
    msg: String
  },
  setup() {
    const isOpen = ref(false);

    return { isOpen };
  },
  data() {
      return {
        showFirst: 'true',
        listUUAA: [],
        listTable: [],
        selectedUUAA: "",
        selectedTable: "",
        file: '',
        data_f: '',
        schema_f: '',
        exampleData: [],
        resultData: "",
        errorData: [],
        schemaName: "",
        selectedEnvi: "raw"
      };
    },
    created() {
        this.loadAllUUAA();
    },
    methods: {
      loadAllUUAA() {
        axios
            .get("http://127.0.0.1:5000/api/v1/schemas/uuaa", )
            .then((res) => {
              this.listUUAA = res.data;
              console.log(this.listUUAA);
            });
      },
      onChangeUUAA() {
      axios
        .get(
          `http://127.0.0.1:5000//api/v1/schemas/uuaa_tables/${this.selectedUUAA}`,
        )
        .then((res) => {
              this.listTable = res.data;
              console.log(this.listTable);
        });
      },
      handleFileUpload( event ){
        this.file = event.target.files[0];
      },
       handleDataUpload( event ){
        this.data_f = event.target.files[0];
      },
       handleSchemaUpload( event ){
        this.schema_f = event.target.files[0];
      },
      submitArtifactory(){
        let formData = new FormData();
        formData.append('data', this.file);
        formData.append('vars', `{"uuaa": "${this.selectedUUAA}", "table": "${this.selectedTable}", "table_env":"${this.selectedEnvi}"}`);
        axios.post( 'http://127.0.0.1:5000/api/v1/schemas/validateartifactory',
                formData,
                {
                headers: {
                    'Content-Type': 'multipart/form-data'
                }
              }
            ).then((res) => {
          console.log(res);
          this.isOpen = true;
          this.exampleData = JSON.parse(res.data.example);
          this.errorData = JSON.parse(res.data.errors);
          this.resultData = res.data.result;
          this.schemaName = `${this.selectedEnvi} - ${this.selectedTable}.output.schema`
          console.log(this.exampleData);
          console.log(this.resultData);
          console.log(this.errorData);
        })
        .catch(function(){
          console.log('FAILURE!!');
        });
      },
      submitWhitSchema(){
        let formData = new FormData();
        formData.append('data', this.data_f);
        formData.append('schema', this.schema_f);
        axios.post( 'http://127.0.0.1:5000/api/v1/schemas/validate',
                formData,
                {
                headers: {
                    'Content-Type': 'multipart/form-data'
                }
              }
            ).then((res) => {
          console.log(res);
          this.isOpen = true;
          this.exampleData = JSON.parse(res.data.example);
          this.errorData = JSON.parse(res.data.errors);
          this.resultData = res.data.result;
          this.schemaName = `${this.schema_f.name}`
          console.log(this.exampleData);
          console.log(this.resultData);
          console.log(this.errorData);
        })
        .catch(function(){
          console.log('FAILURE!!');
        });
      },
      openModal() {
        this.modalOpen = !this.modalOpen;
      }
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
