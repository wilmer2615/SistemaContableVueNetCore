<template>
    <v-layout align-start>
        <v-flex>
            <v-toolbar flat color="white">
                <v-toolbar-title>Clientes</v-toolbar-title>
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
                            <v-flex xs12 sm6 md6>
                                <v-select v-model="tipoDocumento" :items="documentos" label="Tipo Documento"></v-select>
                            </v-flex>
                            <v-flex xs12 sm6 md6>
                                <v-text-field  v-model="numDocumento" label="Numero documento"></v-text-field>
                            </v-flex>
                            <v-flex xs12 sm12 md12>
                                <v-text-field  v-model="direccion" label="Direccion"></v-text-field>
                            </v-flex>
                            <v-flex xs12 sm6 md6>
                                <v-text-field  v-model="telefono" label="Telefono"></v-text-field>
                            </v-flex>
                            <v-flex xs12 sm6 md6>
                                <v-text-field  v-model="email" label="Email"></v-text-field>
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
                        <v-btn color="blue darken-1" text @click.native="close">Cancelar</v-btn>
                        <v-btn color="blue darken-1" text @click.native="guardar">Guardar</v-btn>
                        </v-card-actions>
                    </v-card>
                </v-dialog>
            </v-toolbar>
            <v-data-table
                :headers="headers"
                :items="clientes"
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
                    </td>
                </template>

                <template v-slot:items="props">
                <td>{‌{ props.item.nombre }}</td>
                <td>{‌{ props.item.tipoPersona }}</td>
                <td>{‌{ props.item.tipoDocumento }}</td>
                <td>{‌{ props.item.numDocumento }}</td>
                <td>{‌{ props.item.direccion }}</td>
                <td >{‌{ props.item.telefono }}</td>
                <td >{‌{ props.item.email }}</td>
                {{props}}
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
            clientes:[],
            dialog: false,
            dialogDelete: false,
            headers: [            
                { text: 'Opciones', value: 'opciones', sortable: false },
                { text: 'Nombre', value: 'nombre' },
                { text: 'Tipo Persona', value: 'tipoPersona' },
                { text: 'Tipo documento', value: 'tipoDocumento' },
                { text: 'Numero documento', value: 'numDocumento',sortable: false  },
                { text: 'Direccion', value: 'direccion',sortable: false  },
                { text: 'Telefono', value: 'telefono',sortable: false  },
                { text: 'Email', value: 'email',sortable: false  }

            ],
            search:'',
            editedIndex: -1,
            id:"",
            nombre:"",
            tipoDocumento:"",
            documentos:["Cedula", "T.I","Pasaporte","R.C"],
            numDocumento:"",
            direccion:"",
            telefono:"",
            email:"",
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
        return this.editedIndex === -1 ? 'Nuevo Cliente' : 'Actualizar Cliente'
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
            axios.get('api/Personas/ListarClientes').then(function(response){                
                //console.log(response);
                me.clientes = response.data;
            }).catch(function(error){
                console.log(error);
            });
        }, 
      

        editItem (item) {
            this.id = item.idpersona;
            this.nombre = item.nombre;
            this.tipoDocumento = item.tipoDocumento;
            this.numDocumento = item.numDocumento;
            this.direccion = item.direccion;
            this.telefono = item.telefono;
            this.email = item.email;
            this.editedIndex = 1;
            this.dialog = true
        },
      

        close () {
            this.dialog = false;
            this.limpiar();
        },

        
        limpiar(){
            this.id= "";
            this.nombre="";
            this.numDocumento="";
            this.tipoDocumento="";
            this.direccion="";
            this.telefono="";
            this.email="";
            this.editedIndex = -1;
        },

        guardar () {

            if(this.validar()){
                 return;
            }
            if (this.editedIndex > -1) {
                //codigo para editar
                let me=this;
               
                axios.put('api/Personas/Actualizar',{
                    
                    'idpersona':me.id,
                    'tipoPersona':'Cliente',
                    'nombre':me.nombre,
                    'tipoDocumento':me.tipoDocumento,
                    'numDocumento':me.numDocumento,
                    'direccion':me.direccion,
                    'telefono':me.telefono,
                    'email':me.email
                    }).then(function(response){
                    me.close();
                    me.listar();
                    me.limpiar();
                }).catch(function(error){
                    console.log(error);
                });

            } else {
                //codigo para guardar
                let me=this;
                axios.post('api/Personas/Crear',{

                    'tipoPersona':'Cliente',
                    'nombre':me.nombre,
                    'tipoDocumento':me.tipoDocumento,
                    'numDocumento':me.numDocumento,
                    'direccion':me.direccion,
                    'telefono':me.telefono,
                    'email':me.email
                    }).then(function(response){
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
            if(this.nombre.length<3 || this.nombre.length > 100){
                this.validaMensaje.push("EL nombre debe tener entre 3 y 100 caracteres.");
            }
            if (!this.tipoDocumento){
                this.validaMensaje.push("Seleccione un tipo de documento.");
            } 
            if(this.validaMensaje.length){
                this.valida = 1;
            }
            return this.valida;
        }
      
    }
}
</script>
