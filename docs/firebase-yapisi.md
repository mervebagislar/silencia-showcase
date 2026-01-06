### 📁 Koleksiyon Yapısı
```text
users/
  └── {userId}/
      ├── name: string
      ├── email: string
      ├── photoUrl: string
      ├── createdAt: timestamp
      ├── learningStats/
      │   ├── totalWords: number
      │   ├── totalMinutes: number
      │   └── streak: number
      └── preferences/
          ├── theme: string
          └── notifications: boolean

words/
  └── {wordId}/
      ├── title: string
      ├── category: string
      ├── videoUrl: string
      ├── description: string
      └── createdAt: timestamp

favorites/
  └── {userId}/
      └── {wordId}/
          ├── addedAt: timestamp
          └── wordRef: reference

chats/
  └── {chatId}/
      ├── participants: array
      ├── lastMessage: string
      ├── lastMessageTime: timestamp
      └── messages/
          └── {messageId}/
              ├── senderId: string
              ├── text: string
              ├── timestamp: timestamp
              └── type: string

badges/
  └── {badgeId}/
      ├── title: string
      ├── description: string
      ├── iconUrl: string
      ├── requirement: number
      └── type: string

userBadges/
  └── {userId}/
      └── {badgeId}/
          └── earnedAt: timestamp

progress/
  └── {userId}/
      └── {date}/
          ├── wordsLearned: number
          ├── minutesSpent: number
          └── quizScore: number
```

### 🔐 Firestore Security Rules (Örnek)
```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    
    // Kullanıcılar sadece kendi verilerine erişebilir
    match /users/{userId} {
      allow read: if request.auth != null;
      allow write: if request.auth.uid == userId;
    }
    
    // Kelimeler herkese açık (read), sadece admin yazabilir
    match /words/{wordId} {
      allow read: if true;
      allow write: if request.auth.token.admin == true;
    }
    
    // Favoriler kullanıcıya özel
    match /favorites/{userId}/{wordId} {
      allow read, write: if request.auth.uid == userId;
    }
    
    // Chat mesajları sadece katılımcılara açık
    match /chats/{chatId} {
      allow read: if request.auth.uid in resource.data.participants;
      allow write: if request.auth.uid in resource.data.participants;
      
      match /messages/{messageId} {
        allow read: if request.auth.uid in get(/databases/$(database)/documents/chats/$(chatId)).data.participants;
        allow create: if request.auth.uid in get(/databases/$(database)/documents/chats/$(chatId)).data.participants;
      }
    }
  }
}
```