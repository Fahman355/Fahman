# Fahman
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Chat App</title>
    <style>
        body {
            background-color: #9b59b6; /* Purple Neon Background */
            font-family: Arial, sans-serif; /* Normal Font */
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
        }

        .chat-container {
            background-color: #2c3e50; /* Darker Background for Chat Container */
            color: #ecf0f1; /* Text Color */
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
            width: 80%;
            max-width: 600px;
        }
    </style>
</head>
<body>
    <div id="root"></div>
    <script src="https://www.gstatic.com/firebasejs/8.6.8/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.6.8/firebase-auth.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.6.8/firebase-firestore.js"></script>
    <script>
        // Firebase Configuration
        const firebaseConfig = {
            apiKey: "YOUR_API_KEY",
            authDomain: "YOUR_AUTH_DOMAIN",
            projectId: "YOUR_PROJECT_ID",
            storageBucket: "YOUR_STORAGE_BUCKET",
            messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
            appId: "YOUR_APP_ID"
        };

        // Initialize Firebase
        firebase.initializeApp(firebaseConfig);
        const auth = firebase.auth();
        const db = firebase.firestore();

        // React Code
        class App extends React.Component {
            constructor(props) {
                super(props);
                this.state = {
                    user: null,
                    email: '',
                    password: '',
                    messages: [],
                    input: ''
                };
            }

            componentDidMount() {
                auth.onAuthStateChanged(user => {
                    this.setState({ user });
                    if (user) {
                        this.subscribeToMessages();
                    }
                });
            }

            subscribeToMessages = () => {
                db.collection('messages').orderBy('timestamp').onSnapshot(snapshot => {
                    const messages = snapshot.docs.map(doc => doc.data());
                    this.setState({ messages });
                });
            };

            handleLogin = () => {
                const { email, password } = this.state;
                auth.signInWithEmailAndPassword(email, password).catch(error => console.error(error));
            };

            handleRegister = () => {
                const { email, password } = this.state;
                auth.createUserWithEmailAndPassword(email, password).catch(error => console.error(error));
            };

            handleSendMessage = () => {
                const { input } = this.state;
                if (input.trim()) {
                    db.collection('messages').add({
                        text: input,
                        timestamp: firebase.firestore.FieldValue.serverTimestamp()
                    });
                    this.setState({ input: '' });
                }
            };

            render() {
                const { user, email, password, messages, input } = this.state;

                if (!user) {
                    return (
                        <div className="chat-container">
                            <h2>Login</h2>
                            <input
                                type="email"
                                placeholder="Email"
                                value={email}
                                onChange={(e) => this.setState({ email: e.target.value })}
                            />
                            <input
                                type="password"
                                placeholder="Password"
                                value={password}
                                onChange={(e) => this.setState({ password: e.target.value })}
                            />
                            <button onClick={this.handleLogin}>Login</button>
                            <button onClick={this.handleRegister}>Register</button>
                        </div>
                    );
                }

                return (
                    <div className="chat-container">
                        <h2>Chat</h2>
                        <div className="messages">
                            {messages.map((msg, index) => (
                                <div key={index} className="message">
                                    {msg.text}
                                </div>
                            ))}
                        </div>
                        <input
                            type="text"
                            value={input}
                            onChange={(e) => this.setState({ input: e.target.value })}
                            placeholder="Type your message..."
                        />
                        <button onClick={this.handleSendMessage}>Send</button>
                    </div>
                );
            }
        }

        ReactDOM.render(<App />, document.getElementById('root'));
    </script>
    <script src="https://unpkg.com/react/umd/react.development.js"></script>
    <script src="https://unpkg.com/react-dom/umd/react-dom.development.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
</body>
</html>
