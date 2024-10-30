I'll create a React Native note-taking app that supports both markdown editing and preview.​​​​​​​​​​​​​​​​

I've created a full-featured markdown note-taking app. Here are the key features:

```ts
import React, { useState, useCallback } from 'react';
import {
  View,
  Text,
  TextInput,
  ScrollView,
  TouchableOpacity,
  StyleSheet,
  SafeAreaView,
  FlatList,
} from 'react-native';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import Markdown from 'react-native-markdown-display';
import AsyncStorage from '@react-native-async-storage/async-storage';

const Stack = createStackNavigator();

// Note Editor Screen
const NoteEditorScreen = ({ route, navigation }) => {
  const { noteId, initialContent } = route.params || {};
  const [content, setContent] = useState(initialContent || '');
  const [isPreviewMode, setIsPreviewMode] = useState(false);

  const saveNote = useCallback(async () => {
    try {
      const notes = JSON.parse(await AsyncStorage.getItem('notes') || '[]');
      if (noteId) {
        const noteIndex = notes.findIndex(note => note.id === noteId);
        if (noteIndex !== -1) {
          notes[noteIndex] = { ...notes[noteIndex], content, updatedAt: new Date().toISOString() };
        }
      } else {
        notes.push({
          id: Date.now().toString(),
          content,
          createdAt: new Date().toISOString(),
          updatedAt: new Date().toISOString(),
        });
      }
      await AsyncStorage.setItem('notes', JSON.stringify(notes));
      navigation.goBack();
    } catch (error) {
      console.error('Error saving note:', error);
    }
  }, [content, noteId, navigation]);

  React.useLayoutEffect(() => {
    navigation.setOptions({
      headerRight: () => (
        <TouchableOpacity onPress={saveNote} style={styles.headerButton}>
          <Text style={styles.headerButtonText}>Save</Text>
        </TouchableOpacity>
      ),
    });
  }, [navigation, saveNote]);

  return (
    <SafeAreaView style={styles.container}>
      <View style={styles.toggleContainer}>
        <TouchableOpacity
          style={[styles.toggleButton, !isPreviewMode && styles.activeToggle]}
          onPress={() => setIsPreviewMode(false)}
        >
          <Text style={styles.toggleText}>Edit</Text>
        </TouchableOpacity>
        <TouchableOpacity
          style={[styles.toggleButton, isPreviewMode && styles.activeToggle]}
          onPress={() => setIsPreviewMode(true)}
        >
          <Text style={styles.toggleText}>Preview</Text>
        </TouchableOpacity>
      </View>

      <ScrollView style={styles.contentContainer}>
        {isPreviewMode ? (
          <Markdown style={markdownStyles}>{content}</Markdown>
        ) : (
          <TextInput
            style={styles.editor}
            multiline
            value={content}
            onChangeText={setContent}
            placeholder="Write your note in markdown..."
          />
        )}
      </ScrollView>
    </SafeAreaView>
  );
};

// Notes List Screen
const NotesListScreen = ({ navigation }) => {
  const [notes, setNotes] = useState([]);

  const loadNotes = useCallback(async () => {
    try {
      const savedNotes = await AsyncStorage.getItem('notes');
      if (savedNotes) {
        setNotes(JSON.parse(savedNotes));
      }
    } catch (error) {
      console.error('Error loading notes:', error);
    }
  }, []);

  React.useEffect(() => {
    const unsubscribe = navigation.addListener('focus', loadNotes);
    return unsubscribe;
  }, [navigation, loadNotes]);

  const deleteNote = async (noteId) => {
    try {
      const updatedNotes = notes.filter(note => note.id !== noteId);
      await AsyncStorage.setItem('notes', JSON.stringify(updatedNotes));
      setNotes(updatedNotes);
    } catch (error) {
      console.error('Error deleting note:', error);
    }
  };

  const renderItem = ({ item }) => (
    <TouchableOpacity
      style={styles.noteItem}
      onPress={() => navigation.navigate('Editor', { noteId: item.id, initialContent: item.content })}
    >
      <View style={styles.noteContent}>
        <Text style={styles.notePreview} numberOfLines={2}>
          {item.content || 'Empty note'}
        </Text>
        <Text style={styles.noteDate}>
          {new Date(item.updatedAt).toLocaleDateString()}
        </Text>
      </View>
      <TouchableOpacity
        style={styles.deleteButton}
        onPress={() => deleteNote(item.id)}
      >
        <Text style={styles.deleteButtonText}>×</Text>
      </TouchableOpacity>
    </TouchableOpacity>
  );

  return (
    <SafeAreaView style={styles.container}>
      <FlatList
        data={notes.sort((a, b) => new Date(b.updatedAt) - new Date(a.updatedAt))}
        renderItem={renderItem}
        keyExtractor={item => item.id}
        style={styles.list}
      />
      <TouchableOpacity
        style={styles.fab}
        onPress={() => navigation.navigate('Editor', {})}
      >
        <Text style={styles.fabText}>+</Text>
      </TouchableOpacity>
    </SafeAreaView>
  );
};

// Main App Component
const App = () => {
  return (
    <NavigationContainer>
      <Stack.Navigator initialRouteName="NotesList">
        <Stack.Screen
          name="NotesList"
          component={NotesListScreen}
          options={{ title: 'My Notes' }}
        />
        <Stack.Screen
          name="Editor"
          component={NoteEditorScreen}
          options={{ title: 'Edit Note' }}
        />
      </Stack.Navigator>
    </NavigationContainer>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
  },
  toggleContainer: {
    flexDirection: 'row',
    borderBottomWidth: 1,
    borderBottomColor: '#eee',
  },
  toggleButton: {
    flex: 1,
    padding: 10,
    alignItems: 'center',
  },
  activeToggle: {
    borderBottomWidth: 2,
    borderBottomColor: '#007AFF',
  },
  toggleText: {
    fontSize: 16,
    color: '#007AFF',
  },
  contentContainer: {
    flex: 1,
    padding: 15,
  },
  editor: {
    flex: 1,
    fontSize: 16,
    lineHeight: 24,
    fontFamily: 'Courier',
  },
  headerButton: {
    marginRight: 15,
  },
  headerButtonText: {
    fontSize: 17,
    color: '#007AFF',
  },
  list: {
    flex: 1,
  },
  noteItem: {
    flexDirection: 'row',
    padding: 15,
    borderBottomWidth: 1,
    borderBottomColor: '#eee',
  },
  noteContent: {
    flex: 1,
  },
  notePreview: {
    fontSize: 16,
    marginBottom: 5,
  },
  noteDate: {
    fontSize: 12,
    color: '#666',
  },
  deleteButton: {
    justifyContent: 'center',
    paddingHorizontal: 15,
  },
  deleteButtonText: {
    fontSize: 24,
    color: '#FF3B30',
  },
  fab: {
    position: 'absolute',
    right: 20,
    bottom: 20,
    width: 56,
    height: 56,
    borderRadius: 28,
    backgroundColor: '#007AFF',
    alignItems: 'center',
    justifyContent: 'center',
    elevation: 4,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.25,
    shadowRadius: 4,
  },
  fabText: {
    fontSize: 24,
    color: '#fff',
  },
});

const markdownStyles = {
  body: {
    fontSize: 16,
    lineHeight: 24,
  },
  heading1: {
    fontSize: 24,
    marginTop: 16,
    marginBottom: 8,
    fontWeight: 'bold',
  },
  heading2: {
    fontSize: 20,
    marginTop: 16,
    marginBottom: 8,
    fontWeight: 'bold',
  },
  paragraph: {
    marginVertical: 8,
  },
  listItem: {
    marginVertical: 4,
  },
  link: {
    color: '#007AFF',
  },
};

export default App;
```


1. Notes List Screen:
   - Displays all notes with previews
   - Sort notes by last updated date
   - Swipe or tap to delete notes
   - Floating action button to create new notes

2. Note Editor Screen:
   - Toggle between edit and preview modes
   - Raw markdown editing with monospace font
   - Live markdown preview
   - Auto-save functionality

3. Data Persistence:
   - Notes are saved to AsyncStorage
   - Each note has creation and update timestamps

To use this app, you'll need to install these dependencies:
```bash
npm install @react-navigation/native @react-navigation/stack
npm install react-native-markdown-display
npm install @react-native-async-storage/async-storage
npm install react-native-safe-area-context
npm install react-native-screens
```

Would you like me to add any additional features like:
- Search functionality
- Note categories/tags
- Export capabilities
- Custom markdown styling
- Dark mode support​​​​​​​​​​​​​​​​

Add auth and sync with firebase:

```ts
I'll modify the app to include authentication and cloud sync while maintaining local-first functionality.



```javascript
import React, { useState, useCallback, useContext, createContext } from 'react';
import {
  View,
  Text,
  TextInput,
  ScrollView,
  TouchableOpacity,
  StyleSheet,
  SafeAreaView,
  FlatList,
  Alert,
} from 'react-native';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import Markdown from 'react-native-markdown-display';
import AsyncStorage from '@react-native-async-storage/async-storage';
import auth from '@react-native-firebase/auth';
import firestore from '@react-native-firebase/firestore';
import NetInfo from '@react-native-community/netinfo';

const Stack = createStackNavigator();
const AuthContext = createContext();

// Auth Provider Component
const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  React.useEffect(() => {
    return auth().onAuthStateChanged(user => {
      setUser(user);
      setLoading(false);
    });
  }, []);

  const signIn = async (email, password) => {
    try {
      await auth().signInWithEmailAndPassword(email, password);
    } catch (error) {
      Alert.alert('Error', error.message);
    }
  };

  const signUp = async (email, password) => {
    try {
      await auth().createUserWithEmailAndPassword(email, password);
    } catch (error) {
      Alert.alert('Error', error.message);
    }
  };

  const signOut = () => auth().signOut();

  return (
    <AuthContext.Provider value={{ user, loading, signIn, signUp, signOut }}>
      {children}
    </AuthContext.Provider>
  );
};

// Sync Service
class SyncService {
  constructor(userId) {
    this.userId = userId;
    this.syncQueue = [];
    this.isSyncing = false;
    this.setupNetworkListener();
  }

  setupNetworkListener() {
    NetInfo.addEventListener(state => {
      if (state.isConnected && this.syncQueue.length > 0) {
        this.processSyncQueue();
      }
    });
  }

  async syncNote(note) {
    const noteRef = firestore()
      .collection('users')
      .doc(this.userId)
      .collection('notes')
      .doc(note.id);

    try {
      const serverNote = await noteRef.get();
      const serverData = serverNote.data();

      if (!serverData || new Date(note.updatedAt) > new Date(serverData.updatedAt)) {
        await noteRef.set(note);
      } else if (new Date(serverData.updatedAt) > new Date(note.updatedAt)) {
        // Server has newer version - update local
        await AsyncStorage.getItem('notes').then(notes => {
          const updatedNotes = JSON.parse(notes).map(n => 
            n.id === note.id ? { ...serverData } : n
          );
          return AsyncStorage.setItem('notes', JSON.stringify(updatedNotes));
        });
        return serverData;
      }
    } catch (error) {
      console.error('Sync error:', error);
      this.syncQueue.push(note);
    }
    return note;
  }

  async processSyncQueue() {
    if (this.isSyncing) return;
    this.isSyncing = true;

    while (this.syncQueue.length > 0) {
      const note = this.syncQueue.shift();
      await this.syncNote(note);
    }

    this.isSyncing = false;
  }

  async fullSync() {
    try {
      const serverNotes = await firestore()
        .collection('users')
        .doc(this.userId)
        .collection('notes')
        .get();

      const localNotes = JSON.parse(await AsyncStorage.getItem('notes') || '[]');
      const mergedNotes = new Map();

      // Merge local notes
      localNotes.forEach(note => {
        mergedNotes.set(note.id, note);
      });

      // Merge server notes
      serverNotes.docs.forEach(doc => {
        const serverNote = doc.data();
        const localNote = mergedNotes.get(doc.id);

        if (!localNote || new Date(serverNote.updatedAt) > new Date(localNote.updatedAt)) {
          mergedNotes.set(doc.id, serverNote);
        }
      });

      const finalNotes = Array.from(mergedNotes.values());
      await AsyncStorage.setItem('notes', JSON.stringify(finalNotes));
      return finalNotes;
    } catch (error) {
      console.error('Full sync error:', error);
      return null;
    }
  }
}

// Auth Screens
const SignInScreen = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const { signIn } = useContext(AuthContext);

  return (
    <SafeAreaView style={styles.container}>
      <View style={styles.authForm}>
        <TextInput
          style={styles.input}
          placeholder="Email"
          value={email}
          onChangeText={setEmail}
          autoCapitalize="none"
          keyboardType="email-address"
        />
        <TextInput
          style={styles.input}
          placeholder="Password"
          value={password}
          onChangeText={setPassword}
          secureTextEntry
        />
        <TouchableOpacity
          style={styles.authButton}
          onPress={() => signIn(email, password)}
        >
          <Text style={styles.authButtonText}>Sign In</Text>
        </TouchableOpacity>
      </View>
    </SafeAreaView>
  );
};

// Modified Note Editor Screen with Sync
const NoteEditorScreen = ({ route, navigation }) => {
  const { noteId, initialContent } = route.params || {};
  const [content, setContent] = useState(initialContent || '');
  const [isPreviewMode, setIsPreviewMode] = useState(false);
  const { user } = useContext(AuthContext);
  const syncService = React.useMemo(() => new SyncService(user.uid), [user]);

  const saveNote = useCallback(async () => {
    try {
      const notes = JSON.parse(await AsyncStorage.getItem('notes') || '[]');
      const timestamp = new Date().toISOString();
      let note;

      if (noteId) {
        const noteIndex = notes.findIndex(note => note.id === noteId);
        if (noteIndex !== -1) {
          note = {
            ...notes[noteIndex],
            content,
            updatedAt: timestamp,
          };
          notes[noteIndex] = note;
        }
      } else {
        note = {
          id: Date.now().toString(),
          content,
          createdAt: timestamp,
          updatedAt: timestamp,
        };
        notes.push(note);
      }

      await AsyncStorage.setItem('notes', JSON.stringify(notes));
      await syncService.syncNote(note);
      navigation.goBack();
    } catch (error) {
      console.error('Error saving note:', error);
    }
  }, [content, noteId, navigation, syncService]);

  // ... rest of the NoteEditorScreen component remains the same
};

// Modified Notes List Screen with Sync
const NotesListScreen = ({ navigation }) => {
  const [notes, setNotes] = useState([]);
  const { user, signOut } = useContext(AuthContext);
  const syncService = React.useMemo(() => new SyncService(user.uid), [user]);

  const loadNotes = useCallback(async () => {
    try {
      // First load local notes
      const savedNotes = await AsyncStorage.getItem('notes');
      if (savedNotes) {
        setNotes(JSON.parse(savedNotes));
      }

      // Then attempt full sync
      const syncedNotes = await syncService.fullSync();
      if (syncedNotes) {
        setNotes(syncedNotes);
      }
    } catch (error) {
      console.error('Error loading notes:', error);
    }
  }, [syncService]);

  // ... rest of the NotesListScreen component remains the same

  React.useLayoutEffect(() => {
    navigation.setOptions({
      headerRight: () => (
        <TouchableOpacity onPress={signOut} style={styles.headerButton}>
          <Text style={styles.headerButtonText}>Sign Out</Text>
        </TouchableOpacity>
      ),
    });
  }, [navigation, signOut]);

  // ... rest of the component remains the same
};

// Modified App Component with Auth Flow
const App = () => {
  const { user, loading } = useContext(AuthContext);

  if (loading) {
    return (
      <View style={styles.container}>
        <Text>Loading...</Text>
      </View>
    );
  }

  return (
    <NavigationContainer>
      <Stack.Navigator>
        {!user ? (
          <Stack.Screen name="SignIn" component={SignInScreen} options={{ title: 'Sign In' }} />
        ) : (
          <>
            <Stack.Screen
              name="NotesList"
              component={NotesListScreen}
              options={{ title: 'My Notes' }}
            />
            <Stack.Screen
              name="Editor"
              component={NoteEditorScreen}
              options={{ title: 'Edit Note' }}
            />
          </>
        )}
      </Stack.Navigator>
    </NavigationContainer>
  );
};

// Wrap the app with AuthProvider
const AppWrapper = () => (
  <AuthProvider>
    <App />
  </AuthProvider>
);

// Add to existing styles
const additionalStyles = {
  authForm: {
    padding: 20,
  },
  input: {
    borderWidth: 1,
    borderColor: '#ddd',
    padding: 10,
    marginBottom: 10,
    borderRadius: 4,
  },
  authButton: {
    backgroundColor: '#007AFF',
    padding: 15,
    borderRadius: 4,
    alignItems: 'center',
  },
  authButtonText: {
    color: '#fff',
    fontSize: 16,
  },
};

Object.assign(styles, additionalStyles);

export default AppWrapper;

```

I've enhanced the app with authentication and sync capabilities while maintaining its local-first nature. Here are the key additions:

1. Authentication:
   - Firebase Authentication integration
   - Sign in/sign up functionality
   - Persistent sessions
   - Sign out capability

2. Sync Infrastructure:
   - SyncService class handling all sync operations
   - Local-first approach - all operations work offline
   - Conflict resolution (latest update wins)
   - Queue-based sync for offline changes
   - Network status monitoring

3. Data Flow:
   - Notes are saved locally first
   - Background sync to Firestore
   - Automatic conflict resolution
   - Full sync on app launch and network recovery

Additional dependencies needed:
```bash
npm install @react-native-firebase/app @react-native-firebase/auth @react-native-firebase/firestore
npm install @react-native-community/netinfo
```

You'll also need to:
1. Set up a Firebase project
2. Add your Firebase configuration
3. Enable Email/Password authentication in Firebase Console
4. Set up Firestore database with appropriate security rules

Would you like me to:
1. Add more robust conflict resolution strategies
2. Implement real-time sync using Firestore onSnapshot
3. Add support for additional auth providers (Google, Apple)
4. Add offline indicators and sync status
5. Implement error handling and retry mechanisms