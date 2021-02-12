<template>
  <v-container>
    <v-row justify="center" class="text-center">
      <v-col cols="6">
        <v-form ref="form" v-model="valid" lazy-validation>
          <v-text-field
            v-model="nombre"
            :rules="nameRules"
            label="Nombre del objeto"
            required
          ></v-text-field>

          <v-btn color="success" class="mr-4" @click="crearObjeto()">
            Crear objeto
          </v-btn>
          <v-btn
            :loading="loadRestaurar"
            dark
            color="purple"
            class="mr-4"
            @click="restaurarObjetos()"
          >
            Restaurar
          </v-btn>

          <v-row justify="center" class="mt-3 text-center">
            <v-col cols="4">
              <v-btn
                :loading="loadReplicar"
                dark
                color="blue"
                class="mr-4"
                @click="replicarObjetos()"
              >
                Replicar
              </v-btn>
            </v-col>
            <v-col cols="4">
              <v-select
                :rules="selectRules"
                required
                :items="replicas"
                label="Tipo de réplica"
                v-model="tipoReplica"
              ></v-select>
            </v-col>
          </v-row>
        </v-form>

        <v-data-table
          :headers="headers"
          :items="objetos"
          class="elevation-1 mt-5"
        >
          <template v-slot:item.actions="{ item }">
            <v-icon
              color="red"
              small
              class="mr-2"
              @click="eliminarObjeto(item)"
            >
              mdi-delete
            </v-icon>
          </template>
        </v-data-table>
        <v-alert class="mt-3" color="blue" dark v-show="showAlertMessage">{{
          alertMessage
        }}</v-alert>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from "axios";
export default {
  name: "HelloWorld",

  data: () => ({
    valid: true,
    nombre: "",
    nameRules: [
      (v) => !!v || "El nombre es obligatorio",
      (v) => (v && v.length <= 15) || "Name must be less than 10 characters",
    ],
    selectRules: [(v) => !!v || "El tipo de réplica es obligatorio"],
    dialog: false,
    headers: [
      { text: "Eliminar", value: "actions", sortable: false, align: "center" },
      { text: "Nombre", value: "nombre.$t", align: "center" },
      { text: "Fecha", value: "fecha.$t", align: "center" },
    ],
    objetos: [
      // {
      //   nombre: {
      //     $t: "Objetico 1",
      //   },
      //   fecha: {
      //     $t: "12-03-2021",
      //   },
      // },
      // {
      //   nombre: {
      //     $t: "Objetico 2",
      //   },
      //   fecha: {
      //     $t: "12-03-2021",
      //   },
      // },
    ],
    replicas: ["COMMIT", "ABORT"],
    tipoReplica: "COMMIT",
    loadRestaurar: false,
    loadReplicar: false,
    alertMessage: "Nada",
    showAlertMessage: false,
  }),
  methods: {
    validate() {
      return this.$refs.form.validate();
    },
    async getObjetos() {
      let obj = await axios.get("objeto/consultar");
      this.objetos = obj.data;
      console.log("Se trae esto: ", obj.data);
    },
    async eliminarObjeto(item) {
      console.log("Item: ", item);
      await axios.delete("objeto/eliminar", { data: item });
      this.getObjetos();
    },
    async crearObjeto() {
      if (this.validate()) {
        let date = new Date();
        let obj = {
          nombre: {
            $t: this.nombre,
          },
          fecha: {
            $t: `${date.getDate()}-${date.getMonth() +
              1}-${date.getFullYear()}`,
          },
          accion: {
            $t: "COMMIT",
          },
        };
        await axios.post("objeto/crear", obj);
        this.getObjetos();
      }
    },
    async restaurarObjetos() {
      this.loadRestaurar = true;
      let obj = await axios.get("objeto/restaurar");
      this.loadRestaurar = false;

      console.log("me llega; ", obj);

      if (obj.data.message === "No se restauró") {
        this.alertMessage =
          "No se pudo restaurar la información porque ningún servidor de réplica está activo.";
        this.showAlertMessage = true;
        setTimeout(() => {
          this.showAlertMessage = false;
        }, 5000);
      } else {
        if (obj.data.data.data) {
          this.objetos = obj.data.data.data;
        } else if (obj.data.data) this.objetos = obj.data.data;

        this.alertMessage = "Se restauró la información correctamente";
        this.showAlertMessage = true;
        setTimeout(() => {
          this.showAlertMessage = false;
        }, 5000);
      }

      console.log("Se restaura: ", obj);
    },
    async replicarObjetos() {
      this.loadReplicar = true;

      console.log("TIPO: ", this.tipoReplica);
      let res = await axios.post("objeto/replicar", {
        accion: this.tipoReplica,
      });
      console.log("Resultado de la replicacion: ", res);
      this.alertMessage = res.data.message;
      this.showAlertMessage = true;
      setTimeout(() => {
        this.showAlertMessage = false;
      }, 3000);
      this.loadReplicar = false;
    },
  },

  mounted() {
    this.getObjetos();
  },
};
</script>
