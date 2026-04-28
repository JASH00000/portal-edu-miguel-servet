const firebaseConfig = {
  apiKey: "AIzaSyD3Ofk89GCH_kjyuY6HjsHK6Uy_53qH-W4",
  authDomain: "portal-edu-miguel-servet.firebaseapp.com",
  projectId: "portal-edu-miguel-servet",
  storageBucket: "portal-edu-miguel-servet.firebasestorage.app",
  messagingSenderId: "1068914789060",
  appId: "1:1068914789060:web:d8fb19e8934aec34941791"
};

const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);

const subjects = [
  "Lenguaje",
  "Sociales",
  "Ciencias",
  "Ingles",
  "Seminario",
  "OPV",
  "Matematicas"
];

const grades = [
  "Noveno Grado",
  "1° Bachillerato",
  "2° Bachillerato"
];

export default function App() {
  const [user, setUser] = useState(null);
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [role, setRole] = useState("student");
  const [materials, setMaterials] = useState([]);
  const [selectedGrade, setSelectedGrade] = useState("Noveno Grado");
  const [selectedSubject, setSelectedSubject] = useState("Lenguaje");
  const [text, setText] = useState("");
  const [link, setLink] = useState("");

  useEffect(() => {
    const unsubscribe = onAuthStateChanged(auth, (currentUser) => {
      setUser(currentUser);
      if (currentUser) loadMaterials();
    });

    return () => unsubscribe();
  }, []);

  const registerUser = async () => {
    try {
      await createUserWithEmailAndPassword(auth, email, password);
      alert("Usuario creado correctamente");
    } catch (error) {
      alert(error.message);
    }
  };

  const loginUser = async () => {
    try {
      await signInWithEmailAndPassword(auth, email, password);
    } catch (error) {
      alert("Error al iniciar sesión");
    }
  };

  const loadMaterials = async () => {
    const querySnapshot = await getDocs(collection(db, "materials"));
    const list = [];

    querySnapshot.forEach((doc) => {
      list.push(doc.data());
    });

    setMaterials(list);
  };

  const addMaterial = async () => {
    if (role !== "teacher") return;

    if (!text && !link) return;

    await addDoc(collection(db, "materials"), {
      grade: selectedGrade,
      subject: selectedSubject,
      text: text,
      link: link
    });

    setText("");
    setLink("");

    loadMaterials();
  };

  if (!user) {
    return (
      <div style={{ padding: "40px", textAlign: "center" }}>
        <h1>Portal edu Miguel Servet</h1>

        <input
          placeholder="Correo electrónico"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
        />

        <br /><br />

        <input
          type="password"
          placeholder="Contraseña"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
        />

        <br /><br />

        <select onChange={(e) => setRole(e.target.value)}>
          <option value="student">Estudiante</option>
          <option value="teacher">Docente</option>
        </select>

        <br /><br />

        <button onClick={loginUser}>
          Iniciar Sesión
        </button>

        <button onClick={registerUser}>
          Crear Usuario
        </button>
      </div>
    );
  }

  return (
    <div style={{ padding: "20px" }}>
      <h1>Portal edu Miguel Servet</h1>

      {grades.map((grade) => (
        <div key={grade}>
          <h2>{grade}</h2>

          {subjects.map((subject) => (
            <div key={subject}>
              <h3>{subject}</h3>

              {materials
                .filter(
                  (m) =>
                    m.grade === grade &&
                    m.subject === subject
                )
                .map((item, i) => (
                  <div key={i}>
                    <p>{item.text}</p>

                    {item.link && (
                      <a href={item.link} target="_blank">
                        Abrir material
                      </a>
                    )}
                  </div>
                ))}

              {role === "teacher" && (
                <div>
                  <input
                    placeholder="Texto del material"
                    value={text}
                    onChange={(e) => {
                      setSelectedSubject(subject);
                      setSelectedGrade(grade);
                      setText(e.target.value);
                    }}
                  />

                  <input
                    placeholder="Link del material"
                    value={link}
                    onChange={(e) => setLink(e.target.value)}
                  />

                  <button onClick={addMaterial}>
                    Subir Material
                  </button>
                </div>
              )}
            </div>
          ))}
        </div>
      ))}
    </div>
  );
}