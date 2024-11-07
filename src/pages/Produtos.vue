<template>
    <!-- Criar botão para abrir o modal -->
    <v-container>
        <v-row>
            <v-col class="d-flex justify-end">
                <v-btn color="primary" @click="dialog = !dialog">Cadastrar</v-btn>
            </v-col>
        </v-row>
    </v-container>

    <!-- Criar tabela -->
    <v-row>
        <v-col>
            <div class="table-container">
                <v-table>
                    <thead>
                        <tr>
                            <th>Nome</th>
                            <th>Preço</th>
                            <th>Ação</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="(product, index) in products" :key="product._id">
                            <td>{{ product.nome }}</td>
                            <td>{{ product.preco }}</td>
                            <td>
                                <v-btn color="primary" @click="openEdit(product)">Editar</v-btn>
                                <v-btn color="error" @click="deleteProduct(product._id)">Excluir</v-btn>
                            </td>
                        </tr>
                    </tbody>
                </v-table>
            </div>
        </v-col>
    </v-row>

    <!-- Modal de Cadastro -->
    <v-dialog v-model="dialog">
        <v-card>
            <v-card-title>
                {{ isEdit ? 'Editar Produto' : 'Cadastrar Produto' }}
            </v-card-title>
            <v-card-text>
                <v-form>
                    <v-text-field label="Nome do produto" v-model="productName" :value="productName"></v-text-field>
                    <v-text-field label="Preço do produto" v-model="productPrice" :value="productPrice" type="number"></v-text-field>
                </v-form>
            </v-card-text>
            <v-card-actions>
                <v-btn @click="saveProduct">Salvar</v-btn>
                <v-btn @click="dialog = false">Cancelar</v-btn>
            </v-card-actions>
        </v-card>
    </v-dialog>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';

const products = ref([]);
const editId = ref(null);
const productName = ref('');
const productPrice = ref('');
const isEdit = ref(false);

// Função para obter os produtos da API
const getProducts = async () => {
    try {
        const response = await axios.get('https://cruddesafio-cruddesafios-projects.vercel.app/products/');
        products.value = response.data;
    } catch (error) {
        console.error(error);
    }
};

// Função para excluir um produto
const deleteProduct = async (id) => {
    try {
        await axios.delete(`https://cruddesafio-cruddesafios-projects.vercel.app/products/${id}`);
        getProducts();
    } catch (error) {
        console.error(error);
    }
};

// Função para salvar um produto (POST ou PUT)
const saveProduct = async () => {
    try {
        if (isEdit.value) {
            await axios.put(`https://cruddesafio-cruddesafios-projects.vercel.app/products/${editId.value}`, {
                nome: productName.value,
                preco: productPrice.value
            });
        } else {
            await axios.post('https://cruddesafio-cruddesafios-projects.vercel.app/products/', {
                nome: productName.value,
                preco: productPrice.value
            });
        }
        // Atualiza a lista de produtos
        getProducts();
        // Fecha o modal e limpa os campos
        dialog.value = false;
        productName.value = '';
        productPrice.value = '';
        editId.value = null;
    } catch (error) {
        console.error(error);
    }
};

// Função para abrir o modal de edição
const openEdit = (product) => {
    isEdit.value = true;
    productName.value = product.nome;
    productPrice.value = product.preco;
    editId.value = product._id;
    dialog.value = true;
};

// Carregar os produtos ao montar o componente
onMounted(getProducts);

// Estado do modal
const dialog = ref(false);
</script>

<style scoped>
.table-container {
    border: 1px solid #ccc;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    border-radius: 8px;
    overflow: hidden;
}
</style>
