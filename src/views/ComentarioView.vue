<script setup>
import { ref, onMounted } from 'vue';

const comentarios = ref([]);
const trabalhos = ref([]);
const loading = ref(true);
const formError = ref('');
const successMessage = ref('');
const filtro = ref({ // Novo estado para os filtros de busca
    tituloTrabalho: '',
    urgencia: 1
});
const isFiltered = ref(false); // Indica se a tabela está mostrando resultados filtrados
const apiBaseUrl = 'http://localhost:8080/comentario';
const trabalhoApiUrl = 'http://localhost:8080/trabalho';

// --- ATUALIZAÇÃO 1: Adicionando dataHoraCriacao ao modelo ---
const novoComentarioModelo = {
    conteudo: '',
    dataHoraCriacao: '', // Novo campo para data e hora
    urgencia: 1,
    trabalho: { id: null }
};
// Inicializa o novo objeto com a data/hora atual por padrão, formatada para o input datetime-local
const formatarDataLocal = (date) => {
    const d = new Date(date.getTime() - (date.getTimezoneOffset() * 60000)).toISOString().slice(0, 16);
    return d;
};
const novoComentario = ref({ 
    ...novoComentarioModelo,
    dataHoraCriacao: formatarDataLocal(new Date())
});
// -----------------------------------------------------------------

const calcularSituacao = (comentario) => {
    const dataCriacaoStr = comentario.dataHoraCriacao;
    const urgencia = comentario.urgencia;

    if (!dataCriacaoStr) {
        return 'Data Indisponível';
    }

    const dataCriacao = new Date(dataCriacaoStr);
    
    const dezDiasAtras = new Date();
    dezDiasAtras.setDate(dezDiasAtras.getDate() - 10);

    const isMaisDeDezDias = dataCriacao < dezDiasAtras;
    const isUrgencia3 = urgencia === 3;

    if (isMaisDeDezDias || isUrgencia3) {
        return 'Atenção';
    } else {
        return 'Ok';
    }
};

const limparFormulario = () => {
    // Garante que o campo de data seja resetado para a hora atual, e não vazio
    novoComentario.value = { 
        ...novoComentarioModelo,
        dataHoraCriacao: formatarDataLocal(new Date()) 
    };
    formError.value = '';
    // Limpar mensagem de sucesso após 3 segundos
    setTimeout(() => {
        successMessage.value = '';
    }, 3000);
};

const buscarComentarios = async () => {
    isFiltered.value = false; // Garante que a flag de filtro está desligada na busca total
    try {
        loading.value = true;
        const response = await fetch(apiBaseUrl);
        if (!response.ok) {
            throw new Error(`Erro HTTP: ${response.status}`);
        }
        const data = await response.json();
        comentarios.value = data;
    } catch (error) {
        console.error("Erro ao buscar comentários:", error);
    } finally {
        loading.value = false;
    }
};

const buscarComentariosFiltrados = async () => {
    formError.value = '';
    successMessage.value = '';

    if (!filtro.value.tituloTrabalho.trim() || !filtro.value.urgencia) {
        formError.value = 'Preencha o Título do Trabalho e a Urgência para buscar.';
        return;
    }

    // Construção da URL de busca com Query Parameters
    const buscaUrl = `${apiBaseUrl}/busca?trabalho=${encodeURIComponent(filtro.value.tituloTrabalho.trim())}&urgencia=${filtro.value.urgencia}`;
    
    try {
        loading.value = true;
        const response = await fetch(buscaUrl);
        if (!response.ok) {
            throw new Error(`Erro HTTP: ${response.status}`);
        }
        
        const data = await response.json();
        comentarios.value = data; // Atualiza a tabela com os resultados
        isFiltered.value = true; // Liga a flag de filtro
    } catch (error) {
        console.error("Erro ao buscar comentários filtrados:", error);
        formError.value = error.message || 'Erro ao buscar os comentários.';
        comentarios.value = []; // Limpa a tabela em caso de erro
        isFiltered.value = true;
    } finally {
        loading.value = false;
    }
};

const limparFiltro = () => {
    filtro.value = { tituloTrabalho: '', urgencia: 1 };
    formError.value = '';
    buscarComentarios(); // Volta a exibir todos os comentários
};

const buscarTrabalhos = async () => {
    try {
        const response = await fetch(trabalhoApiUrl);
        if (!response.ok) {
            throw new Error(`Erro ao buscar trabalhos: ${response.status}`);
        }
        trabalhos.value = await response.json();
    } catch (error) {
        console.error("Erro ao buscar lista de trabalhos:", error);
    }
};

const salvarComentario = async () => {
    formError.value = '';
    successMessage.value = '';

    // Validação de Frontend
    if (!novoComentario.value.conteudo.trim()) {
        formError.value = 'O campo Conteúdo é obrigatório.';
        return;
    }
    if (novoComentario.value.urgencia < 1 || novoComentario.value.urgencia > 3) {
        formError.value = 'A urgência deve ser 1, 2 ou 3.';
        return;
    }
    if (!novoComentario.value.trabalho.id) {
        formError.value = 'A seleção do Trabalho é obrigatória.';
        return;
    }
    // Nova validação para o campo de data
    if (!novoComentario.value.dataHoraCriacao) {
        formError.value = 'O campo Data/Hora de Criação é obrigatório.';
        return;
    }

    // O Spring Boot aceita o formato ISO 8601. O input datetime-local já fornece esse formato.
    // O backend já tem lógica para setar a data se for null, mas estamos enviando explicitamente aqui.

    try {
        loading.value = true;
        const response = await fetch(apiBaseUrl, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(novoComentario.value)
        });

        if (!response.ok) {
            const errorData = await response.json();
            throw new Error(errorData.message || `Falha no cadastro com status: ${response.status}`);
        }

        const novoItem = await response.json();
        
        comentarios.value.unshift(novoItem);

        successMessage.value = 'Comentário cadastrado com sucesso!';
        limparFormulario();
        
    } catch (error) {
        console.error("Erro ao salvar comentário:", error);
        formError.value = error.message || 'Erro desconhecido ao cadastrar o comentário.';
    } finally {
        loading.value = false;
    }
};

onMounted(() => {
    buscarComentarios();
    buscarTrabalhos();
});
</script>

<template>
  <div class="comentarios-container">

    <!-- Formulário de Consulta (EXISTENTE) -->
    <div class="form-card consulta-card">
        <h3>Consultar Comentários por Filtro</h3>
        <form @submit.prevent="buscarComentariosFiltrados" class="comentario-form">
            <div class="form-row">
                <div class="form-group">
                    <label for="filtro-titulo">Título do Trabalho</label>
                    <input type="text" id="filtro-titulo" v-model="filtro.tituloTrabalho" required>
                </div>
                
                <div class="form-group urgencia-filtro">
                    <label for="filtro-urgencia">Urgência (1 a 3)</label>
                    <input type="number" id="filtro-urgencia" v-model.number="filtro.urgencia" min="1" max="3" required>
                </div>
            </div>

            <p v-if="formError" class="error-message">{{ formError }}</p>

            <div class="form-actions">
                <button type="submit" :disabled="loading" class="submit-button search-button">
                    Buscar
                </button>
                <button type="button" @click="limparFiltro" class="submit-button clear-button">
                    Limpar Busca
                </button>
            </div>
        </form>
    </div>

    <!-- Formulário de Cadastro de Novo Comentário -->
    <div class="form-card cadastro-card">
        <h3>Novo Comentário</h3>
        
        <form @submit.prevent="salvarComentario" class="comentario-form">
            <div class="form-group">
                <label for="conteudo">Conteúdo (Obrigatório)</label>
                <textarea id="conteudo" v-model="novoComentario.conteudo" rows="3" required></textarea>
            </div>

            <div class="form-row three-columns">
                <!-- CAMPO DATA E HORA DE CRIAÇÃO (NOVO) -->
                <div class="form-group">
                    <label for="dataHoraCriacao">Data/Hora de Criação</label>
                    <input type="datetime-local" id="dataHoraCriacao" v-model="novoComentario.dataHoraCriacao" required>
                </div>
                <!-- FIM DO CAMPO NOVO -->

                <div class="form-group">
                    <label for="urgencia">Urgência (1 a 3)</label>
                    <input type="number" id="urgencia" v-model.number="novoComentario.urgencia" min="1" max="3" required>
                </div>
                
                <div class="form-group">
                    <label for="trabalho">Trabalho Associado (Obrigatório)</label>
                    <select id="trabalho" v-model.number="novoComentario.trabalho.id" required>
                        <option :value="null" disabled>Selecione um Trabalho</option>
                        <option v-for="trabalho in trabalhos" :key="trabalho.id" :value="trabalho.id">
                            {{ trabalho.titulo }}
                        </option>
                    </select>
                </div>
            </div>

            <p v-if="successMessage" class="success-message">{{ successMessage }}</p>
            
            <button type="submit" :disabled="loading" class="submit-button">
                {{ loading ? 'Salvando...' : 'Cadastrar Comentário' }}
            </button>
        </form>
    </div>

    <!-- Tabela de Comentários (EXISTENTE) -->
    <h2 class="table-title">Tabela de Comentários</h2>
    
    <p v-if="loading">Carregando comentários...</p>
    
    <p v-else-if="!loading && comentarios.length === 0">
        {{ isFiltered ? 'Nenhum registro foi encontrado para os critérios fornecidos.' : 'Nenhum comentário cadastrado no momento.' }}
    </p>

    <table v-else class="comentarios-tabela">
      <thead>
        <tr>
          <th>ID</th>
          <th>Conteúdo</th>
          <th>Título do Trabalho</th>
          <th>Situação</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="comentario in comentarios" :key="comentario.id">
          <td>{{ comentario.id }}</td>
          <td>{{ comentario.conteudo }}</td>
          <td>{{ comentario.trabalho ? comentario.trabalho.titulo : 'N/A' }}</td>
          <td>
            <strong>{{ calcularSituacao(comentario) }}</strong>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<style scoped>
/* CORES DO DARK MODE */
:root {
    --bg-dark: #1e1e1e;
    --bg-card: #282c34;
    --text-light: #cccccc;
    --border-dark: #444;
    --border-light: #666;
}

.comentarios-container {
    padding: 20px;
    background-color: var(--bg-dark);
    color: var(--text-light);
    border-radius: 8px;
    box-shadow: 0 4px 15px rgba(0, 0, 0, 0.4);
}

.form-card {
    background: var(--bg-card);
    padding: 20px;
    margin-bottom: 30px;
    border-radius: 6px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

.form-card h3 {
    margin-top: 0;
    color: var(--text-light);
    border-bottom: 1px solid var(--border-dark);
    padding-bottom: 10px;
}

.comentario-form {
    display: flex;
    flex-direction: column;
    gap: 15px;
}

.form-group label {
    display: block;
    margin-bottom: 5px;
    font-weight: bold;
    color: var(--text-light);
}

.form-group input, .form-group textarea, .form-group select {
    width: 100%;
    padding: 10px;
    border: 1px solid var(--border-light);
    border-radius: 4px;
    box-sizing: border-box;
    font-size: 14px;
    background-color: #3c4046;
    color: var(--text-light);
}

.form-group textarea {
    resize: vertical;
}

.form-row {
    display: flex;
    gap: 20px;
}

/* Novo estilo para a linha de três colunas no cadastro */
.form-row.three-columns > .form-group {
    flex: 1;
}

.form-actions {
    display: flex;
    gap: 10px;
    justify-content: flex-start;
}

/* MENSAGENS E CORES DE FEEDBACK */
.error-message {
    color: #ffcccc;
    background-color: #610b0b;
    border: 1px solid #dc3545;
    padding: 10px;
    border-radius: 4px;
    font-weight: 500;
}

.success-message {
    color: #c3e6cb;
    background-color: #218838; 
    border: 1px solid #155724;
    padding: 10px;
    border-radius: 4px;
    font-weight: 500;
}

/* BOTÕES - Cores mantidas para contraste */
.submit-button {
    padding: 10px 15px;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-weight: bold;
    transition: background-color 0.3s;
}

.search-button {
    background-color: #ffc107;
    color: #333;
}
.search-button:hover:not(:disabled) {
    background-color: #e0a800;
}

.clear-button {
    background-color: #6c757d;
}
.clear-button:hover:not(:disabled) {
    background-color: #5a6268;
}

.submit-button:not(.search-button):not(.clear-button) {
    background-color: #007bff;
}

.submit-button:hover:not(:disabled):not(.search-button):not(.clear-button) {
    background-color: #0056b3;
}

.submit-button:disabled {
    background-color: #495057;
    cursor: not-allowed;
}

.table-title {
    margin-top: 40px;
    color: var(--text-light);
    border-bottom: 2px solid var(--border-dark);
    padding-bottom: 10px;
}

/* ESTILOS DA TABELA */
.comentarios-tabela {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
  background-color: var(--bg-card);
}
.comentarios-tabela th, .comentarios-tabela td {
  border: 1px solid var(--border-dark);
  padding: 12px;
  text-align: left;
  color: var(--text-light);
}
.comentarios-tabela th {
  background-color: #3c4046;
  font-weight: bold;
  color: white;
}
</style>