<template>
  <div>
    <h2 class="text-center">Formulário de Notas</h2>
    <!--********************************Editar**************************************-->
     <!--1º Formulário de Editar notas -->
    <form @submit.prevent="editarNota(nota)" v-if="acaoEditar"><!--SE -> acaoEditar = verdadeiro chamar a ação Editar -->
     <h4>Editar Nota</h4>
      <input type="text" placeholder="Seu nome" class="form-control mb-2" v-model="nota.name" />
      <input
        type="text" placeholder="Descrição..." class="form-control mb-2" v-model="nota.description" />
      <button class="btn btn-success" type="submit">Salvar</button>
      <button class="btn btn-danger" type="submit" @click="cancelarEdicao()">Cancelar</button>
    </form>

    <!--********************************Salvar**************************************-->
    <!--2º  Formulário de Salvar notas -->
    <form @submit.prevent="adicionar" v-else><!--Senão -> acaoEditar = Falso chamar a ação Adicionar -->
      <h4>Adicionar Nota</h4>
      <input type="text" placeholder="Seu nome" class="form-control mb-2" v-model="nota.name" />
      <input
        type="text" placeholder="Descrição..." class="form-control mb-2" v-model="nota.description" />
      <button class="btn btn-primary" type="submit">Adicionar</button>
    </form>

    <!--********************************Listar notas**************************************-->
    <hr class="mt-3">  
    <h3 class="text-center">Lista de notas</h3>
           <table class="table table-hover table-striped">
            <thead>
                <tr>
                    <th scope="row">Nome</th>
                    <th scope="row">Descrição</th>
                    <th scope="row">
                      Ações  
                    </th>
                </tr>
            </thead>
            <tbody>
                
                <tr v-for="(item, index) in notas" :key="index">             
                <td>{{item.name}}</td>
                <td>{{item.description}}</td>
                    <td width="10px">
                        <a href="#" class="btn btn-warning" @click="editarFormulario(item)">Editar</a>
                    </td>
                    <td width="10px">
                        <a href="#" class="btn btn-danger" @click="excluirNota(item, index)">Excluir</a>
                    </td>
                    
                </tr>
            </tbody>
        </table>
  </div>
</template>
<!--********************************Fim do template**************************************-->
<script>
export default {
  data() {
    return {
      notas: [],//assim que salvar recebemos a resposta (this.notas.push)
      nota: { name: "", description: "" },

      //Quando clicar no botão editar e acaoEditar: muda para verdadeiro
      acaoEditar: false
    };
  },

  created(){
      //acessa o metódo index para listar todas as notas
      axios.get('/notas')
        .then(resp =>{
            this.notas = resp.data;
        })
    
  },    
  //********************Inicio dos Métodos********************
  methods: {
    adicionar() {
      if (this.nota.name.trim() === "" || this.nota.description=== "") {

        alert("Você deve preencher todos os campos!");
        return;
      }
      //console.log(this.nota.name, this.nota.description);

      const parametros = {
        name: this.nota.name,
        description: this.nota.description
      }

      this.nota.name = "";
      this.nota.description = "";

      //acessa o metódo store para salvar todas as notas na bd
      axios.post("/notas", parametros)
        .then(resp => {
            this.notas.push(resp.data)
      })

    },//fim da função adicionar

    //********************Inicio dos metódos de edição********************
    editarFormulario(item){
        this.acaoEditar       = true;
        this.nota.name        = item.name;
        this.nota.description = item.description;
        this.nota.id          = item.id;

    },  

    editarNota(nota){
        const parametros = {
            name: nota.name,
            description: nota.description
        };

        axios.put(`/notas/${nota.id}`, parametros)
            .then(resp =>{
                //formulário volta a ser de adicionar
                this.acaoEditar = false;

                const index = this.notas.findIndex(item => item.id === nota.id);
                this.notas[index] = resp.data;

                //limpa o formulário logo depois    
                this.nota = { name: "", description: "" },

                //refresh na lista 
                axios.get('/notas')
                    .then(resp =>{
                     this.notas = resp.data;
                })
            })    

    },

    cancelarEdicao(){

        //formulário volta a ser de adicionar
        this.acaoEditar = false;

         //limpa o formulário logo depois    
        this.nota = { name: "", description: "" };
    },

    //********************Inicio da função excluirNota()********************
    excluirNota(item, index){
        axios.delete(`/notas/${item.id}`)
        .then(()=>{
            //Na função splice recebe o indice e quantidade de item que serão excluidos callback
            this.notas.splice(index, 1);
        })
    }
  }
};
</script>