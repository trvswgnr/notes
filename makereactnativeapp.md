I'll create a React Native note-taking app that supports both markdown editing and preview.​​​​​​​​​​​​​​​​

I've created a full-featured markdown note-taking app. Here are the key features:

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