<!-- index.html -->
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>CityConnect</title>
  <link rel="stylesheet" href="style.css" />
</head>
<body>
  <header>
    <h1>CityConnect</h1>
    <nav>
      <a href="#">Accueil</a>
      <a href="#">Groupes</a>
      <a href="#">Connexion</a>
    </nav>
  </header>

  <main>
    <h2>Connecte-toi avec les gens autour de toi</h2>
    <p>Découvre et rejoins des groupes dans ta ville. Participe, discute, rencontre !</p>
    <button onclick="window.location.href='inscription.html'">Commencer</button>
  </main>

  <footer>
    <p>&copy; 2025 CityConnect</p>
  </footer>
</body>
</html>
/* style.css */
body {
    margin: 0;
    font-family: 'Segoe UI', sans-serif;
    background: linear-gradient(to bottom right, #83a4d4, #b6fbff);
    color: #333;
  }
  
  header {
    background-color: #fff;
    padding: 20px;
    text-align: center;
    box-shadow: 0 4px 6px rgba(0,0,0,0.1);
  }
  
  header h1 {
    margin: 0;
    color: #3366cc;
  }
  
  nav a {
    margin: 0 15px;
    text-decoration: none;
    color: #3366cc;
    font-weight: bold;
  }
  
  main {
    text-align: center;
    margin-top: 100px;
  }
  
  main button {
    padding: 15px 30px;
    background-color: #3366cc;
    color: #fff;
    border: none;
    border-radius: 10px;
    cursor: pointer;
    font-size: 16px;
  }
  
  main button:hover {
    background-color: #254a99;
  }
  
  footer {
    position: absolute;
    bottom: 10px;
    width: 100%;
    text-align: center;
    font-size: 14px;
  }==

  import { createContext, useState, useEffect } from 'react';
import jwtDecode from 'jwt-decode';

export const AuthContext = createContext();

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);

  useEffect(() => {
    const token = localStorage.getItem('token');
    if (token) {
      const decoded = jwtDecode(token);
      setUser(decoded);
    }
  }, []);

  const login = (token) => {
    localStorage.setItem('token', token);
    const decoded = jwtDecode(token);
    setUser(decoded);
  };

  const logout = () => {
    localStorage.removeItem('token');
    setUser(null);
  };

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
};
import { Link } from 'react-router-dom';
import { useContext } from 'react';
import { AuthContext } from '../context/AuthContext';

const Navbar = () => {
  const { user, logout } = useContext(AuthContext);

  return (
    <nav>
      <Link to="/">Home</Link>
      {user ? (
        <>
          <Link to="/dashboard">Dashboard</Link>
          <button onClick={logout}>Logout</button>
        </>
      ) : (
        <>
          <Link to="/login">Login</Link>
          <Link to="/register">Register</Link>
        </>
      )}
    </nav>
  );
};
import { useContext } from 'react';
import { Navigate } from 'react-router-dom';
import { AuthContext } from '../context/AuthContext';

const PrivateRoute = ({ children }) => {
  const { user } = useContext(AuthContext);
  return user ? children : <Navigate to="/login" />;
};
import { useState } from 'react';
import axios from 'axios';
import { useNavigate } from 'react-router-dom';

const Register = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [username, setUsername] = useState('');
  const navigate = useNavigate();

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      await axios.post('http://localhost:5000/api/auth/register', { username, email, password });
      navigate('/login');
    } catch (err) {
      alert("Erreur d'inscription");
    }
  };
import { useContext } from 'react';
import { AuthContext } from '../context/AuthContext';

const Dashboard = () => {
  const { user } = useContext(AuthContext);

  return (
    <div>
      <h2>Bienvenue, {user?.username || "Utilisateur"}</h2>
      <p>Voici les groupes dans ta zone géographique.</p>
    </div>
  );
};

export default Dashboard;
import { useEffect, useState } from 'react';
import { useParams } from 'react-router-dom';
import axios from 'axios';

const GroupPage = () => {
  const { id } = useParams();
  const [group, setGroup] = useState(null);

  useEffect(() => {
    axios.get(`http://localhost:5000/api/groups/${id}`)
      .then(res => setGroup(res.data))
      .catch(err => console.error(err));
  }, [id]);

  if (!group) return <p>Chargement...</p>;

  return (
    <div>
      <h2>{group.name}</h2>
      <p>{group.description}</p>
    </div>
  );
};

export default GroupPage;
