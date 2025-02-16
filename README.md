import 'package:flutter/material.dart';

import 'controles/controle_planeta.dart';
import 'modelos/planeta.dart';
import 'telas/tela_planeta.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Planetas',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const MyHomePage(title: 'App - Planeta'),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({
    super.key, 
    required this.title});

  final String title;

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  final ControlePlaneta _controlePlaneta = ControlePlaneta();
  List<Planeta> _planetas = [];
  
  get onPressed => null;

  @override
  void initState() {
    super.initState();
    _lerPlanetas();
  }

  Future<void> _lerPlanetas() async {
    final resultado = await _controlePlaneta.lerPlanetas();
    setState(() {
      _planetas = resultado;
    });
  }

  void _incluirPlaneta(BuildContext context) {
    Navigator.push(
      context,
      MaterialPageRoute(
        builder: (context) => TelaPlaneta(
          onFinalizado: () { 
            _lerPlanetas(); },)),
       ).then((_) {
      _lerPlanetas(); // Recarrega a lista de planetas quando voltar da tela de inclusão
    });

  }

@override
Widget build(BuildContext context) {
  return Scaffold(
    appBar: AppBar(
      backgroundColor: Theme.of(context).colorScheme.inversePrimary,
      title: Text(widget.title),
    ),
    body: Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          const Text('Planetas'),
          Expanded(
            child: ListView.builder(
              itemCount: _planetas.length,
              itemBuilder: (context, index) {
                final planeta = _planetas[index];
                return ListTile(
                  title: Text(planeta.nome),
                  subtitle: Text(planeta.apelido!),
                  trailing: IconButton(
                     icon: const Icon(Icons.delete),
                    onPressed: onPressed, 
                  )
                );
              },
            ),
          ),
        ],
      ),
    ),
    floatingActionButton: FloatingActionButton(
      onPressed: () {
        _incluirPlaneta(context);
      },
      child: const Icon(Icons.add),
    ),
  );
}
}































import 'package:flutter/material.dart';
import '../controles/controle_planeta.dart';
import '../modelos/planeta.dart';

class TelaPlaneta extends StatefulWidget {
  final Function() onFinalizado;

  const TelaPlaneta({
    super.key, 
  required this.onFinalizado});

  @override
  State<TelaPlaneta> createState() => _TelaPlanetaState();
}

class _TelaPlanetaState extends State<TelaPlaneta> {
  final _formKey = GlobalKey<FormState>();

  final TextEditingController _nomeController = TextEditingController();
  final TextEditingController _tamanhoController = TextEditingController();
  final TextEditingController _distanciaController = TextEditingController();
  final TextEditingController _apelidoController = TextEditingController();

  final ControlePlaneta _controlePlaneta = ControlePlaneta();
  final Planeta _planeta = Planeta.vazio();

  @override
  void initState() {
    super.initState();
  }

  @override
  void dispose() {
    _nomeController.dispose();
    _tamanhoController.dispose();
    _distanciaController.dispose();
    _apelidoController.dispose();
    super.dispose();
  }

  Future<void> _inserirPlaneta() async {
    await _controlePlaneta.inserirPlaneta(_planeta);
  }

  void _submitForm() {
    if (_formKey.currentState!.validate()) {
      _formKey.currentState!.save();


      _inserirPlaneta();


      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(
          content: Text('Dados do planeta foram salvos com sucesso!'),
        ),
      );
      Navigator.of(context).pop();
      widget.onFinalizado();
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        centerTitle: true,
        title: const Text('Cadastrar Planeta'),
        elevation: 3,
      ),
      body: Padding(
        padding: const EdgeInsets.symmetric(horizontal: 40.0, vertical: 20.0),
        child: Form(
          key: _formKey,
          child: SingleChildScrollView(
            child: Column(
              children: [
                TextFormField(
                  controller: _nomeController,
                  decoration: const InputDecoration(
                    labelText: 'Nome',
                  ),
                  autovalidateMode: AutovalidateMode.onUserInteraction,
                  validator: (value) {
                    if (value == null || value.isEmpty || value.length < 3) {
                      return 'Informe o nome do planeta (3 ou mais caracteres)';
                    }
                    return null;
                  },
                  onSaved: (value) {
                    _planeta.nome = value!;
                  },
                ),
                TextFormField(
                  controller: _tamanhoController,
                  decoration: const InputDecoration(
                    labelText: 'Tamanho (em km)',
                  ),
                  keyboardType: TextInputType.number,
                  autovalidateMode: AutovalidateMode.onUserInteraction,
                  validator: (value) {
                    if (value == null || value.isEmpty) {
                      return 'Informe o tamanho do planeta';
                    }
                    if (double.tryParse(value) == null) {
                      return 'Informe um valor válido';
                    }
                    return null;
                  },
                  onSaved: (value) {
                    _planeta.tamanho = double.parse(value!);
                  },
                ),
                TextFormField(
                  controller: _distanciaController,
                  decoration: const InputDecoration(
                    labelText: 'Distância (em km)',
                  ),
                  keyboardType: TextInputType.number,
                  autovalidateMode: AutovalidateMode.onUserInteraction,
                  validator: (value) {
                    if (value == null || value.isEmpty) {
                      return 'Informe a distância do planeta';
                    }
                    if (double.tryParse(value) == null) {
                      return 'Informe um valor válido';
                    }
                    return null;
                  },
                  onSaved: (value) {
                    _planeta.distancia = double.parse(value!);
                  },
                ),
                TextFormField(
                  controller: _apelidoController,
                  decoration: const InputDecoration(
                    labelText: 'Apelido',
                  ),
                  onSaved: (value) {
                    _planeta.apelido = value!;
                  },
                ),
                const SizedBox(
                  height: 20.0,
                ),
                ElevatedButton(
                  onPressed: _submitForm,
                  child: const Text('Salvar'),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}



























import 'package:path/path.dart';
import 'package:sqflite/sqflite.dart';

import '../modelos/planeta.dart';

class ControlePlaneta {
  static Database? _bd;

  Future<Database> get bd async {
    if (_bd != null) return _bd!;
    _bd = await iniBD('planeta.bd');
    return _bd!;
  }

  Future<Database> iniBD(String localArquivo) async {
    final caminhoBD = await getDatabasesPath();
    final caminho = join(caminhoBD, localArquivo);
    return await openDatabase(
      caminho,
      version: 1,
      onCreate: _criarBD,
    );
  }

  Future<void> _criarBD(Database bd, int versao) async {
    const sql = '''
    CREATE TABLE planeta (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      nome TEXT NOT NULL,
      tamanho REAL NOT NULL,
      distancia REAL NOT NULL,
      apelido TEXT
    );
    ''';
    await bd.execute(sql);
  }

  Future<List<Planeta>> listarPlanetas() async {
    final db = await bd;
    final resultado = await db.query('planeta');
    return resultado.map((item) => Planeta.fromMap(item)).toList();
  }

  Future<int> inserirPlaneta(Planeta planeta) async {
    final db = await bd;
    return await db.insert('planeta', planeta.toMap());
  }

  Future<List<Planeta>> lerPlanetas() async {
    final db = await bd;
    final resultado = await db.query('planeta');
    return resultado.map((item) => Planeta.fromMap(item)).toList();
  }
}





















class Planeta {
  int? id;
  String nome;
  double tamanho;
  double distancia;
  String? apelido;

// Construtor da classe Planeta
  Planeta({
   this.id,
    required this.nome,
    required this.tamanho,
    required this.distancia,
    this.apelido,
  });
//construtor alternativo
  Planeta.vazio()
       : nome = '',
        tamanho = 0.0,
        distancia = 0.0,
        apelido = '';

  factory Planeta.fromMap(Map<String, dynamic> map) {
    return Planeta(
      id: map['id'],
      nome: map['nome'],
      tamanho: map['tamanho'],
      distancia: map['distancia'],
      apelido: map['apelido'],
    );
  }
