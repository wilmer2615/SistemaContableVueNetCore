<template>
    <v-layout align-start>
        <v-flex>
            <v-toolbar flat color="white">
                <v-toolbar-title>Usuarios</v-toolbar-title>
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
                            <v-flex xs12 sm6 md6>
                                <v-text-field v-model="nombre" label="Nombre"></v-text-field>
                            </v-flex>
                            <v-flex xs12 sm6 md6>
                                <v-select v-model="idrol" :items="roles" label="Rol"></v-select>
                            </v-flex>
                            <v-flex xs12 sm6 md6>
                                <v-select v-model="tipoDocumento" :items="documentos" label="Tipo Documento"></v-select>
                            </v-flex>
                            <v-flex xs12 sm6 md6>
                                <v-text-field  v-model="numDocumento" label="Numero documento"></v-text-field>
                            </v-flex>
                            <v-flex xs12 sm6 md6>
                                <v-text-field  v-model="direccion" label="Direccion"></v-text-field>
                            </v-flex>
                            <v-flex xs12 sm6 md6>
                                <v-text-field  v-model="telefono" label="Telefono"></v-text-field>
                            </v-flex>
                            <v-flex xs12 sm6 md6>
                                <v-text-field  v-model="email" label="Email"></v-text-field>
                            </v-flex>
                            <v-flex xs12 sm6 md6>
                                <v-text-field type="password" v-model="passWord" label="Password"></v-text-field>
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
                :items="usuarios"
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
                <td>{‌{ props.item.rol }}</td>
                <td>{‌{ props.item.tipoDocumento }}</td>
                <td>{‌{ props.item.numDocumento }}</td>
                <td>{‌{ props.item.direccion }}</td>
                <td >{‌{ props.item.telefono }}</td>
                <td >{‌{ props.item.email }}</td>
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
            usuarios:[],
            dialog: false,
            dialogDelete: false,
            headers: [            
                { text: 'Opciones', value: 'opciones', sortable: false },
                { text: 'Nombre', value: 'nombre' },
                { text: 'Rol', value: 'rol' },
                { text: 'Tipo documento', value: 'tipoDocumento' },
                { text: 'Numero documento', value: 'numDocumento',sortable: false  },
                { text: 'Direccion', value: 'direccion',sortable: false  },
                { text: 'Telefono', value: 'telefono',sortable: false  },
                { text: 'Email', value: 'email',sortable: false  },
                { text: 'Estado', value: 'condicion',sortable: false  },

            ],
            search:'',
            editedIndex: -1,
            id:"",
            idrol:"",
            nombre:"",
            roles:[],
            tipoDocumento:"",
            documentos:["Cedula", "T.I","Pasaporte","R.C"],
            numDocumento:"",
            direccion:"",
            telefono:"",
            email:"",
            passWord:"",
            actPassWord:false,
            passwordAnt:'',
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
        return this.editedIndex === -1 ? 'Nuevo Usuario' : 'Actualizar Usuario'
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
        this.select();
    },

    methods:{

        listar(){
            let me = this;
            axios.get('api/Usuarios/Listar').then(function(response){                
                //console.log(response);
                me.usuarios = response.data;
            }).catch(function(error){
                console.log(error);
            });
        },

          select(){
            let me = this;
            let rolesArray=[];
            axios.get('api/Roles/Select').then(function(response){                
                rolesArray = response.data;
                rolesArray.map(function(elem){
                    me.roles.push({text: elem.nombre, value: elem.idrol})
                });

            }).catch(function(error){
                console.log(error);
            });
        },
      

        editItem (item) {
            this.id = item.idusuario;
            this.idrol = item.idrol,
            this.nombre = item.nombre;
            this.tipoDocumento = item.tipoDocumento;
            this.numDocumento = item.numDocumento;
            this.direccion = item.direccion;
            this.telefono = item.telefono;
            this.email = item.email;
            this.passWord = item.passWordHash;
            this.passwordAnt = item.passWordHash;
            this.editedIndex = 1;
            this.dialog = true
        },
      

        close () {
            this.dialog = false;
            this.limpiar();
        },

        
        limpiar(){
            this.id= "";
            this.idrol="";
            this.nombre="";
            this.numDocumento="";
            this.tipoDocumento="";
            this.direccion="";
            this.telefono="";
            this.email="";
            this.passwordAnt="";
            this.actPassWord=false;
            this.passWord="";
            this.editedIndex = -1;
        },

        guardar () {

            if(this.validar()){
                 return;
            }
            if (this.editedIndex > -1) {
                //codigo para editar
                let me=this;
                if(me.passWord != me.passwordAnt){
                    me.actPassWord=true;
                }
                
                axios.put('api/Usuarios/Actualizar',{
                    
                    'idusuario':me.id,
                    'idrol':me.idrol,
                    'nombre':me.nombre,
                    'tipoDocumento':me.tipoDocumento,
                    'numDocumento':me.numDocumento,
                    'direccion':me.direccion,
                    'telefono':me.telefono,
                    'email':me.email,
                    'passWord':me.passWord,
                    'actPassWord':me.actPassWord
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
                axios.post('api/Usuarios/Crear',{

                    'idrol':me.idrol,
                    'nombre':me.nombre,
                    'tipoDocumento':me.tipoDocumento,
                    'numDocumento':me.numDocumento,
                    'direccion':me.direccion,
                    'telefono':me.telefono,
                    'email':me.email,
                    'password':me.passWord
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
            if (!this.idrol){
                this.validaMensaje.push("Seleccione un rol.");
            }
            if (!this.tipoDocumento){
                this.validaMensaje.push("Seleccione un tipo de documento.");
            } 
            if (!this.email){
                this.validaMensaje.push("Ingrese el email del usuario.");
            }
             if (!this.passWord){
                this.validaMensaje.push("Ingrese el password del usuario.");
            }
            if(this.validaMensaje.length){
                this.valida = 1;
            }
            return this.valida;
        },

        activarDesactivarMostrar(accion, item){
            this.adModal = 1;
            this.adNombre = item.nombre;
            this.adId = item.idusuario;
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
            axios.put('api/Usuarios/Activar/'+ this.adId,{}).then(function(response){
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
            axios.put('api/Usuarios/Desactivar/'+ this.adId,{}).then(function(response){
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
