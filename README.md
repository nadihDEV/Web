# CP05WEB
BACKEND 
import { db } from "../db.js";
//No caso do get não é necessário incluir req de requisição
export const getAgendas = (_, res) => {
const q = "SELECT * FROM agenda ORDER BY nome";
db.query(q, (err, data) => {
if (err) return res.json(err);
return res.status(200).json(data);
});
};
export const addAgenda = (req, res) => {
const q = "INSERT INTO agenda (`nome`, `endereco`,`telefone`,`email`) VALUES(?)";
const values = [
req.body.nome,
req.body.endereco,
req.body.telefone,
req.body.email,
];
db.query(q, [values], (err) => {
if (err) return res.json(err);
return res.status(200).json("Registro criado com sucesso");
});
};
export const updateAgenda = (req, res) => {
const q = "UPDATE tb_agenda SET `nome` = ?, `endereco` = ? ,`telefone` = ?,`email` = ? WHERE `codigo` = ?";const values = [
 req.body.nome,
 req.body.endereco,
req.body.telefone,
req.body.email,
];
 db.query(q, [...values, req.params.codigo], (err) => {
if (err) return res.jason(err);
return res.status(200).json("Registro atualizado com sucesso");
});
};
export const deleteAgenda = (req, res) => {
const q = "DELETE FROM tb_agenda WHERE `codigo` = ?";
db.query(q, [req.params.codigo], (err) => {
if (err) return res.json(err);
return res.status(200).json("Registro deletado com sucesso");
});
};



import express from "express";
import {
getAgendas,
addAgenda, 
updateAgenda, 
deleteAgenda
} from "../controllers/agenda.js";
const router = express.Router();
router.get("/", getAgendas);
router.post("/", addAgenda);
router.put("/:codigo", updateAgenda);
router.delete("/:codigo", deleteAgenda);
export default router;



import mysql from "mysql";
export const db = mysql.createConnection({
host: "localhost",
user: "root",
password: "123456",
database: "bd_agenda",
});



import express from "express";
import agendaRoutes from "./routes/agendas.js";
import cors from "cors";
//Para utilizar o modo json para alterações como
// Post e Put
const app = express();
app.use(express.json());
app.use(cors());
app.use("/", agendaRoutes);
//Escutar a porta 8800
app.listen(8800);



FRONTEND

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>React App</title>
</head>
<body>
<noscript>You need to enable JavaScript to run this app.</noscript>
<div id="root"></div>
</body>
</html>



import React, { useEffect, useRef } from "react";
import styled from "styled-components";
import axios from "axios";
import { toast } from "react-toastify";
const FormContainer = styled.form`
display: flex;
align-items: flex-end;
gap: 10px;
flex-wrap: wrap;
background-color: #fff;
padding:20px;
box-shadow: 0px 0px 5px #ccc;
border-radius: 5px;
`;
const InputArea = styled.div`
display: flex;
flex-direction: column;
`;
const Input = styled.input`
width: 120px;
padding: 0 10px;
border: 1px solid #bbb;
border-radius: 5px;
height: 40px;
`;
const Label = styled.label``; 
const Button = styled.button`
padding: 10px;
cursor: pointer;
border-radius: 5px;
border: none;
background-color: #2c73d2;
color: white;
height: 42px;
`;
const Form = ({getAgendas, onEdit, setOnEdit}) => {
const ref = useRef();
useEffect(() => {
if (onEdit){
const agenda = ref.current;
agenda.nome.value = onEdit.nome;
agenda.endereco.value = onEdit.endereco;
agenda.telefone.value = onEdit.telefone;
agenda.email.value = onEdit.email;
}
}, [onEdit]);
const handleSubmit = async (e) => {
e.preventDefault();
const agenda = ref.current;
if ( !agenda.nome.value || 
!agenda.endereco.value || 
!agenda.telefone.value || 
!agenda.email.value )
{
return toast.warn("Preencha todos os campos");
}
if (onEdit) {
await axios
.put("http://localhost:8800/" + onEdit.codigo, {
nome: agenda.nome.value,
endereco: agenda.endereco.value,
telefone: agenda.telefone.value,
email: agenda.email.value,
})
.then(({ data }) => toast.success(data))
.catch(({ data }) => toast.error(data));
} else {
await axios
.post("http://localhost:8800", {
nome: agenda.nome.value,
endereco: agenda.endereco.value,
telefone: agenda.telefone.value,
email: agenda.email.value,
})
.then(({ data }) => toast.success(data))
.catch(({ data }) => toast.error(data));
}
agenda.nome.value = "";
agenda.endereco.value = "";
agenda.telefone.value = "";
agenda.email.value = "";
setOnEdit(null);
getAgendas();
};
return(
<FormContainer ref={ref} onSubmit={handleSubmit}>
<InputArea> 
<Label> Nome</Label>
<Input name="nome" /> 
</InputArea>
<InputArea> 
<Label> Endereco</Label>
<Input name="endereco" />
</InputArea>
<InputArea> 
<Label> Telefone</Label>
<Input name="telefone" />
</InputArea>
<InputArea> 
<Label> E_mail</Label>
<Input name="email" type="email" />
</InputArea>
<Button type="submit">GRAVAR</Button>
</FormContainer>
);
 };
export default Form;



import React from "react";
import axios from "axios";
import styled from "styled-components";
import { FaTrash, FaEdit} from "react-icons/fa";
import { toast} from "react-toastify";
const Table = styled.table`
width: 100%;
background-color: #fff;
padding: 20px;
box-shadow: 0px 0px 5px #ccc;
border-radius: 5px;
max-width: 1000px;
margin: 20px auto;
word-break: break-all;
`;
export const Thead = styled.thead``;
export const Tbody = styled.tbody``;
export const Tr = styled.tr``;
export const Th = styled.th`
text-align: start;
border-bottom: inset;
padding-bottom: 5px;
@media (max-width: 500px){
${(props) => props.onlyWeb && "display: none"}
}
`;
export const Td = styled.td`
padding-top: 15px;
text-align: ${(props) => (props.alignCenter ? "center" : "start")};
width: ${(props) => (props.width ? props.width : "auto")};
@media (max-width: 800px) {
${(props) => props.onlyWeb && "display: none"}
}
`;
const Grid = ({ agendas, setAgendas, setOnEdit})=> {
const handleEdit = (item) => {
setOnEdit(item);
};
const handleDelete = async (codigo) => {
await axios
.delete("http://localhost:8800/" + codigo)
.then(({ data }) => {
const newArray = agendas.filter((agenda) => agenda.codigo !== 
codigo);
setAgendas(newArray);
toast.success(data);
})
.catch(({ data }) => toast.error(data));
setOnEdit(null);
};
return (
<Table>
<Thead>
<Tr>
<Th>Nome</Th>
<Th>Endereco</Th>
<Th>Telefone</Th>
<Th>Email</Th>
<Th></Th>
<Th></Th>
</Tr>
</Thead>
<Tbody>
{agendas.map((item, i) => (
<Tr key={i}>
<Td width="22%">{item.nome}</Td>
<Td width="22%">{item.endereco}</Td>
<Td width="15%">{item.telefone}</Td>
<Td width="22%">{item.email}</Td>
<Td alignCenter width="4%">
<FaEdit onClick={() => handleEdit(item)}/>
</Td>
<Td alignCenter width="4%">
<FaTrash onClick={() => handleDelete(item.codigo)}/>
</Td>
</Tr>
))}
</Tbody>
</Table>
);
};
export default Grid;



//Importar a biblioteca styled-components associando a createGlobalStyle
import { createGlobalStyle } from "styled-components";
// Criar uma constante com as definições de createGlobalSyle
// Esses definições nada mais são do que o estilo CSS aplicado no JavaScript
// a partir da biblioteca do React
const Global = createGlobalStyle`
* {
margin: 0;
padding: 0;
font-family: sans-serif;
}
body {
width: 100vw; //Para ocupar todo o espaco da tela 
height:100vh;
display:flex;
justify-content:center;
background-color: #f2f2f2;
}
`;
// O estilo definido no arquivo global.js, deve ser exportada a constante 
// global para que o resto do código-fonte tenha acesso
export default Global;


import GlobalSyle from "./style/global";
import { toast, ToastContainer } from "react-toastify";
import "react-toastify/dist/ReactToastify.css"
import styled from "styled-components";
import Form from "./components/Form.js";
import Grid from "./components/Grid";
import { useEffect, useState } from "react";
import axios from "axios";
const Container = styled.div`
width: 100%;
max-width:800px;
margin-top:20px;
display:flex;
flex-direction:column;
align-items:center;
gap: 10px;
`;
const Title = styled.h2 ``;
function App() {
const [agendas, setAgendas] = useState([]);
const [onEdit, setOnEdit] = useState(null);
const getAgendas = async () => {
try {
const res = await axios.get("http://localhost:8800");
setAgendas(res.data.sort((a, b) => (a.nome > b.nome ? 1 : -1)));
} catch (error){
toast.error(error);
}
};
useEffect(() => {
getAgendas();
}, [setAgendas]);
return (
<>
<Container> 
<Title>AGENDA FRONT-END: REACT e BACK-END: NODE com 
MYSQL</Title>
<Form onEdit={onEdit} setOnEdit={setOnEdit} getAgendas={getAgendas}/>
<Grid agendas={agendas} setAgendas={setAgendas} 
setOnEdit={setOnEdit}/>
</Container>
<ToastContainer autoClose = {3000} />
<GlobalSyle /> </>
);
}

export default App;



import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";
const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(
<React.StrictMode>
<App />
</React.StrictMode>
);
