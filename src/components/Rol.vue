<template>
    <v-layout align-start>
        <v-flex>
            <v-toolbar flat color="white">
                <v-toolbar-title>Roles</v-toolbar-title>
                <v-divider
                class="mx-2"
                inset
                vertical
                ></v-divider>
                <v-spacer></v-spacer>
                <v-text-field class="text-xs-center" v-model="search" append-icon="search" label="Búsqueda" single-line hide-details></v-text-field>
                <v-spacer></v-spacer>               
            </v-toolbar>
            <v-data-table
                :headers="headers"
                :items="roles"
                :search="search"
                class="elevation-1"
            >
                 
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
            roles:[],
            dialog: false,
            dialogDelete: false,
            headers: [
                { text: 'Nombre', value: 'nombre' },
                { text: 'Descripcion', value: 'descripcion',sortable: false  },
                { text: 'Estado', value: 'condicion',sortable: false  },

            ],
            search:''
        }   
    },
    computed: {
        
    },

    watch: {
       
    },

    created () {
        this.listar();
    },

    methods:{

        listar(){
            let me = this;
            axios.get('api/Roles/Listar').then(function(response){
                //console.log(response);
                me.roles = response.data;
            }).catch(function(error){
                console.log(error);
            });
        },
        
       

    }
}
</script>
