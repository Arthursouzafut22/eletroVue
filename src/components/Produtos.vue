
<script>
export default {
  data() {
    return {
      produtos: [],
      produto: false,
      carrinho: [],
      carrinhoTotal: 0,
      mensagemAlerta: "item adicionado",
      alertaAtivo: false,
      carrinhoAtivo: false,
    };
  },

  computed: {
    carrinhoTotal() {
      let total = 0;
      if (this.carrinho.length) {
        const fill = this.carrinho;

        for (const filter of fill) {
          total += filter.preco;
        }
      }

      return total;
    },
  },

  methods: {
    fetchProdutos() {
      fetch("/api/produto.json")
        .then((r) => r.json())
        .then((r) => {
          this.produtos = r;
        });
    },
    fetchProduto(id) {
      fetch(`/api/produtos/${id}/dados.json`)
        .then((r) => r.json())
        .then((r) => {
         (this.produto = r);
        });
    },
    abrirModal(id) {
      this.fetchProduto(id);
      window.scrollTo({
        top: 0,
        behavior: "smooth",
      });
    },
    fecharModal({ target, currentTarget }) {
      if (target === currentTarget) {
        this.produto = false;
      }
    },
    clickForaCarrinho({ target, currentTarget }) {
      if (target === currentTarget) {
        this.carrinhoAtivo = false;
      }
    },
    adicionarItem() {
      this.produto[0].estoque--;
      const { nome } = this.produto[0];
      this.carrinho.push(this.produto[0]);
      this.alerta(`${nome} Adicionado ao carrinho`);
    },
    removerItem() {
     this.carrinho.pop();
    },
    compararEstoque() {
      const items = this.carrinho.filter(({id}) => id === this.produto.id);
      this.produto.estoque -= items.length;
    },
    alerta(mensagem) {
      this.mensagemAlerta = mensagem;
      this.alertaAtivo = true;

      setTimeout(() => {
        this.alertaAtivo = false;
      }, 1500);
    },
  },

  created() {
    this.fetchProdutos();
  },
};
</script>

<template>
  <header class="header">
    <img src="/api/imgs/apple-big-logo (1).png" alt="" />
    <div class="carrinho_menu" @click="carrinhoAtivo = true">
      <p>{{ "R$" + carrinhoTotal }} | {{ carrinho.length }}</p>
      <img src="/api/imgs/carrinho-compras.png" alt="" />
    </div>
  </header>

  <section class="produtos">
    <div
      v-for="item in produtos"
      @click="abrirModal(item.id)"
      :key="item.id"
      class="produto"
    >
      <img :src="item.img" :alt="item.nome" class="produto_img" />
      <div class="produto_info">
        <span class="produto_preco">{{ "R$" + item.preco }}</span>
        <h2 class="produto_titulo">{{ item.nome }}</h2>
      </div>
    </div>
  </section>

  <section
    class="modal"
    v-for="product in produto"
    :key="product"
    @click="fecharModal"
  >
    <div class="modal_container">
      <div class="modal_img">
        <img :src="product.img" :alt="product.nome" />
      </div>
      <div class="modal_dados">
        <button class="modal_fechar" @click="produto = false">X</button>
        <span class="modal_preco">{{ "R$" + product.preco }}</span>
        <h2 class="modal_titulo">{{ product.nome }}</h2>
        <p>{{ product.descricao }}</p>

        <button
          class="modal_btn"
          v-if="product.estoque > 0"
          @click="adicionarItem(product.id)"
        >
          Adicionar Item
        </button>
        <button class="modal_btn" v-else disabled>Produto esgotado</button>
      </div>
    </div>
  </section>

  <section
    class="carrinho_modal"
    :class="{ ativo: carrinhoAtivo }"
    @click="clickForaCarrinho"
  >
    <div class="carrinho_container">
      <button class="carrinho_fechar" @click="carrinhoAtivo = false">X</button>
      <h2 class="carrinho_titulo">Carrinho</h2>
      <div v-if="carrinho.length > 0">
        <ul class="carrinho_lista">
          <li v-for="item in carrinho" :key="item" class="carrinho_item">
            <p>{{ item.nome }}</p>
            <p class="carrinho_preco">{{ "R$" + item.preco }}</p>
            <button class="carrinho_remover" @click="removerItem">X</button>
          </li>
        </ul>
        <p class="carrinho_total">{{ "R$" + carrinhoTotal }}</p>
        <button class="carrinho_finalizar">Finalizar Compra</button>
      </div>
      <p v-else>Carrinho Vazio</p>
    </div>
  </section>

  <div class="alerta" :class="{ ativo: alertaAtivo }">
    <p class="alerta_mensagem">{{ mensagemAlerta }}</p>
  </div>
</template>


<style scoped>
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  max-width: 900px;
  padding: 10px 0px;
  margin: 0 auto;
}

.header img {
  max-width: 60px;
}

.carrinho_menu {
  display: flex;
  align-items: center;
}
.carrinho_menu p {
  font-size: 20px;
  font-weight: bold;
}

.carrinho_menu img {
  max-width: 50px;
  display: block;
  cursor: pointer;
}

.produtos {
  max-width: 900px;
  margin: 100px auto 0 auto;
}

.produto {
  display: flex;
  align-items: center;
  margin-bottom: 40px;
  background: #ffffff;
  box-shadow: 0 0 2rem rgba(0, 0, 0, 0.1);
  cursor: pointer;
}

.produto_img {
  max-width: 300px;
  margin-right: 40px;
}

.produto_titulo {
  font-size: 3rem;
  line-height: 1;
}

.produto_preco {
  color: rgb(0, 0, 0, 0, 0.5);
}

.modal::before {
  content: "";
  position: fixed;
  top: 0px;
  left: 0px;
  width: 100%;
  height: 100vh;
  background-color: rgba(0, 0, 0, 0.5);
}

.modal {
  display: flex;
  flex-direction: column;
  align-items: center;
  position: absolute;
  top: 0px;
  left: 0px;
  width: 100%;
  padding: 80px;
}

.modal_container {
  position: relative;
  background: linear-gradient(to right, transparent 250px, white 250px);
  z-index: 1;
  display: grid;
  align-items: end;
  gap: 50px;
  padding: 0px 50px 50px 0px;
  animation: fadeIn2 0.5s forwards;
}

@keyframes fadeIn2 {
  from {
    opacity: 0;
    transform: translate3d(50%, 0, 0);
  }
  to {
    opacity: 1;
    transform: translate3d(0%, 0, 0);
  }
}

.modal_img {
  grid-column: 1;
  box-shadow: 0px 3px 4px rgb(0, 0, 0, 0.1), 0px 4px 10px rgba(0, 0, 0, 0.2);
  margin-top: 50px;
}

.modal_img img {
  grid-column: 1;
  max-width: 380px;
  display: block;
}

.modal_dados {
  grid-column: 2;
  max-width: 600px;
}

.modal_titulo {
  font-size: 3rem;
}

.modal_btn {
  margin-top: 60px;
  background: #000;
  cursor: pointer;
  color: #ffffff;
  border: none;
  font-size: 1rem;
  padding: 16px 28px;
  font-family: Arial, Helvetica, sans-serif;
  border-radius: 10px;
}

.modal_btn:active {
  background-color: #808080;
}

.modal_fechar {
  width: 40px;
  height: 40px;
  position: absolute;
  top: -10px;
  right: -10px;
  font-size: 1rem;
  font-weight: bold;
  border: none;
  border-radius: 10px;
  cursor: pointer;
  background: #000;
  color: #fff;
  box-shadow: 0px 3px 4px rgb(0, 0, 0, 0.1), 0px 4px 10px rgba(0, 0, 0, 0.2);
}

.alerta {
  position: absolute;
  top: 20px;
  left: 0px;
  z-index: 10;
  width: 100%;
  text-align: center;
  display: none;
}

.alerta.ativo {
  display: block;
  animation: fadeIn 0.5s forwards;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(-50%);
  }
  to {
    opacity: 1;
    transform: translateY(0%);
  }
}

.alerta_mensagem {
  background: #ffffff;
  display: inline-block;
  padding: 20px 40px;
  box-shadow: 0px 3px 4px rgb(0, 0, 0, 0.1), 0px 4px 10px rgba(0, 0, 0, 0.2);
}

.carrinho_modal::before {
  content: '';
  position: fixed;
  top: 0px;
  left: 0px;
  width: 100%;
  height: 100vh;
  background-color: rgb(0, 0, 0, 0.5);
}

.carrinho_modal {
  position: absolute;
  display: flex;
  flex-direction: column;
  top: 0px;
  left: 0px;
  width: 100%;
  padding: 20px;
  display: none;
}


.carrinho_modal.ativo {
  display: block;

}

.carrinho_container {
  position: relative;
  margin: 80px auto;
  background: #ffffff;
  padding: 20px;
  max-width: 800px;
  animation: fadeIn 0.5s forwards;
}

.carrinho_item {
  display: grid;
  grid-template-columns: 1fr 1fr 50px;
  border-bottom: 1px solid rgb0(0,0,0,.1);
  padding: 12px 0;
}

.carrinho_titulo {
  margin-bottom: 10px;
  padding-bottom: 20px;
  border-bottom: 2px solid #000000;
}

.carrinho_remover {
  border: none;
  font-size: 1rem;
  font-weight: bold;
  background: none;
  cursor: pointer;
}

.carrinho_preco {
  text-align: right;
}

.carrinho_total {
  text-align: right;
  padding: 10px 50px 10px 0;
  margin-bottom: 20px;
}

.carrinho_fechar {
  width: 40px;
  height: 40px;
  position: absolute;
  top: -10px;
  right: -10px;
  font-size: 1rem;
  font-weight: bold;
  border: none;
  border-radius: 10px;
  cursor: pointer;
  background: #000;
  color: #fff;
  box-shadow: 0px 3px 4px rgb(0, 0, 0, 0.1), 0px 4px 10px rgba(0, 0, 0, 0.2);
}

.carrinho_finalizar {
  display: block;
  margin-left: auto;
  background: #000;
  cursor: pointer;
  color: #ffffff;
  font-size: 1rem;
  padding: 10px 20px;
  border: none;

}



@media screen and (max-width:900px) {
  #app {
    padding: 0 10px;
  }
  .produtos {
    margin-top: 40px;
  }
  .produto {
    flex-direction: column;
    align-items: flex-start;
    max-width: 300px;
    margin: 30px auto;
  }
  .produto_info {
    padding: 20px;
  }
  .produto_img {
    max-width: 100%;

  }
  .produto_titulo {
    font-size: 1.5rem;
  }
  .modal {
    padding: 15px;
  }
  .modal_container {
    gap: 20px;
    background: #ffffff;
    padding: 10px 0;
    margin-top: 20px;
    
  }
  .modal_img {
    grid-row: 2;
  }
  .modal_img img {
    width: 100%;
    max-width: initial;
  }
  .modal_dados {
    grid-column: 1;
    padding: 12px;
  }
  .modal_btn {
    margin-top: 20px;
  }








}









</style>