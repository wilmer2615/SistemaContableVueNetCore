<template>
    <v-layout align-start>
        <v-flex>
            <v-toolbar flat color="white">
                <v-toolbar-title>Categorías</v-toolbar-title>
                <v-divider
                class="mx-2"
                inset
                vertical
                ></v-divider>
                <v-spacer></v-spacer>
                <v-text-field class="text-xs-center" v-model="search" append-icon="search" label="Búsqueda" single-line hide-details></v-text-field>
                <v-spacer></v-spacer>
                <v-dialog v-model="dialog" max-width="500px">
                    <template v-slot:activator="{ on }">    
                        <v-btn color="primary" dark class="mb-2" v-on="on">Nuevo</v-btn>
                    </template>
                    <v-card>
                        <v-card-title>
                        <span class="headline">{{ formTitle }}</span>
                        </v-card-title>
            
                        <v-card-text>
                        <v-container grid-list-md>
                            <v-layout wrap>
                            <v-flex xs12 sm12 md12>
                                <v-text-field v-model="nombre" label="Nombre"></v-text-field>
                            </v-flex>
                            <v-flex xs12 sm12 md12>
                                <v-text-field v-model="descripcion" label="Descripcion"></v-text-field>
                            </v-flex>
                            <v-flex xs12 sm12 md12 v-show="valida">
                                <div class="red--text" v-for="v in validaMensaje" :key="v" v-text="v">

                                </div>
                            </v-flex>
                            </v-layout>
                        </v-container>
                        </v-card-text>
            
                        <v-card-actions>
                        <v-spacer></v-spacer>
                        <v-btn color="blue darken-1" flat @click.native="close">Cancelar</v-btn>
                        <v-btn color="blue darken-1" flat @click.native="guardar">Guardar</v-btn>
                        </v-card-actions>
                    </v-card>
                </v-dialog>
                <v-dialog v-model="adModal" max-width="290">
                    <v-card>
                        <v-card-title class="headline" v-if="adAccion == 1">¿Activar Item?</v-card-title>
                        <v-card-title class="headline" v-if="adAccion == 2">¿Desactivar Item?</v-card-title>
                        <v-card-text>
                            Estas a punto de
                            <span v-if="adAccion==1">Activar</span>
                            <span v-if="adAccion==2">Desactivar</span>
                            el item {{adNombre}}
                        </v-card-text>
                        <v-card-actions>
                            <v-spacer></v-spacer>
                            <v-btn color="green darken-1" flat="flat" @click="activarDesactivarCerrar">
                                Cancelar
                            </v-btn>
                            <v-btn v-if="adAccion == 1" color="orange darken-1" flat="flat" @click="activar">
                                Activar
                            </v-btn>
                            <v-btn v-if="adAccion == 2" color="orange darken-1" flat="flat" @click="desactivar">
                                Desactivar
                            </v-btn>
                        </v-card-actions>

                    </v-card>
                </v-dialog>
            </v-toolbar>
            <v-data-table
                :headers="headers"
                :items="categorias"
                :search="search"
                class="elevation-1"
            >
                 <template v-slot:[`item.opciones`]="{ item }">
                    <td class="justify-center layout px-0">
                        <v-icon
                        small
                        class="mr-2"
                        @click="editItem(item)"
                        >
                        edit
                        </v-icon>
                    <template v-if="item.condicion == true">
                        <v-icon
                        small
                        @click="activarDesactivarMostrar(2,item)"
                        >
                        block
                        </v-icon>
                    </template>
                      <template v-else>
                        <v-icon
                        small
                        @click="activarDesactivarMostrar(1,item)"
                        >
                        check
                        </v-icon>
                    </template>
                    </td>
                </template>

                <template v-slot:items="props">
                <td>{‌{ props.item.nombre }}</td>
                <td >{‌{ props.item.descripcion }}</td>
                {{props}}
                 </template>
                <template v-slot:[`item.condicion`]="{item}">
                    <v-card-text v-if="item.condicion" class="blue--text">Activo</v-card-text>
                    <v-card-text v-if="!item.condicion" class="red--text">Inactivo</v-card-text>                      
                </template>
 
               
 
                <template v-slot:no-data>
                <v-btn color="primary" @click="listar">Resetear</v-btn>
                </template>
            </v-data-table>
        </v-flex>
    </v-layout>
</template>

<script>
import axios from 'axios'
export default {
    data() {
        return{
            categorias:[],
            dialog: false,
            dialogDelete: false,
            headers: [            
                { text: 'Opciones', value: 'opciones', sortable: false },
                { text: 'Nombre', value: 'nombre' },
                { text: 'Descripcion', value: 'descripcion',sortable: false  },
                { text: 'Estado', value: 'condicion',sortable: false  },

            ],
            search:'',
            editedIndex: -1,
            id:"",
            nombre:"",
            descripcion:"",
            valida: 0,
            validaMensaje:[],
            adModal: 0,
            adAccion: 0,
            adNombre: "",
            adId: ""
  
        }   
    },
    computed: {
        formTitle () {
        return this.editedIndex === -1 ? 'Nueva Categoria' : 'Actualizar Categoria'
        },
    },

    watch: {
        dialog (val) {
        val || this.close()
        },
        dialogDelete (val) {
        val || this.closeDelete()
        },
    },

    created () {
        this.listar();
    },

    methods:{

        listar(){
            let me = this;
            axios.get('api/Categorias/Listar').then(function(response){                
                //console.log(response);
                me.categorias = response.data;
            }).catch(function(error){
                console.log(error);
            });
        },
      

        editItem (item) {
            this.id = item.idcategoria;
            this.nombre = item.nombre;
            this.descripcion = item.descripcion;
            this.editedIndex = 1;
            this.dialog = true
        },

        deleteItem (item) {
            this.editedIndex = this.desserts.indexOf(item)
            this.editedItem = Object.assign({}, item)
            this.dialogDelete = true
        },

        deleteItemConfirm () {
            this.desserts.splice(this.editedIndex, 1)
            this.closeDelete()
        },

        close () {
            this.dialog = false;
            this.limpiar();
        },

        closeDelete () {
            this.dialogDelete = false
            this.$nextTick(() => {
                this.editedItem = Object.assign({}, this.defaultItem)
                this.editedIndex = -1
            })
        },

        limpiar(){
            this.id= "";
            this.nombre="";
            this.descripcion="";
            this.editedIndex = -1;
        },

        guardar () {

            if(this.validar()){
                 return;
            }
            if (this.editedIndex > -1) {
                //codigo para editar

                  let me=this;
                axios.put('api/Categorias/Actualizar',{'nombre':me.nombre, 'descripcion':me.descripcion, 'idcategoria':me.id})
                        .then(function(response){
                    me.close();
                    me.listar();
                    me.limpiar();
                }).catch(function(error){
                    console.log(error);
                });

            } else {
                //codigo para guardar
                let me=this;
                axios.post('api/Categorias/Crear',{'nombre':me.nombre, 'descripcion':me.descripcion}).then(function(response){
                    me.close();
                    me.listar();
                    me.limpiar();
                }).catch(function(error){
                    console.log(error);
                });
            }
        },

        validar(){
            this.valida = 0;
            this.validaMensaje=[];
            if(this.nombre.length<3 || this.nombre.length > 50){
                this.validaMensaje.push("EL nombre debe tener entre 3 y 50 caracteres");
            }
            if(this.validaMensaje.length){
                this.valida = 1;
            }
            return this.valida;
        },

        activarDesactivarMostrar(accion, item){
            this.adModal = 1;
            this.adNombre = item.nombre;
            this.adId = item.idcategoria;
            if(accion == 1){
                this.adAccion = 1;
            }else if(accion == 2 ){
                this.adAccion = 2;
            }else{
                this.admodal = 0;
            }
        },

        activar(){
            let me=this;
            axios.put('api/Categorias/Activar/'+ this.adId,{}).then(function(response){
                me.adModal = 0;
                me.adAccion = 0;
                me.adNombre = "";
                me.adId = "";
                me.listar();
            }).catch(function(error){
                console.log(error);
            });
        },

        desactivar(){
            let me=this;
            axios.put('api/Categorias/Desactivar/'+ this.adId,{}).then(function(response){
            me.adModal = 0;
            me.adAccion = 0;
            me.adNombre = "";
            me.adId = "";
            me.listar();
        }).catch(function(error){
            console.log(error);
        });
        },

        activarDesactivarCerrar(){
            this.adModal = 0;
        }

    }
}
</script>
