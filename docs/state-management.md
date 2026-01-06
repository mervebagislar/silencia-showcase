# 🔄 Silencia – State Management (Riverpod)

Bu doküman, **Silencia** mobil uygulamasında kullanılan **state management**
yaklaşımını ve mimariyle olan ilişkisini açıklamak amacıyla hazırlanmıştır.

Uygulamada **Riverpod 2.x** kullanılmıştır ve state yönetimi,
**Clean Architecture** prensiplerine uygun şekilde kurgulanmıştır.

---

## 🎯 State Management Seçimi

Silencia mobil uygulamasında **Riverpod (2.x)** tercih edilmiştir.

**Tercih edilme nedenleri:**

- Compile-time güvenlik (type-safe)
- Global context ihtiyacı olmadan state yönetimi
- Clean Architecture ile uyum
- Test edilebilir ve ölçeklenebilir yapı
- Async state'lerin (loading / error / data) net yönetimi
- Firebase ve Stream tabanlı yapılarla güçlü entegrasyon

---

## 🧩 Clean Architecture ile İlişkisi

Riverpod, yalnızca **Presentation Layer** içerisinde konumlanır  
ve **Domain Layer** ile **Use Case**'ler üzerinden iletişim kurar.

```text
UI (Widgets)
    ↓
Riverpod Provider
    ↓
Use Case (Domain)
    ↓
Repository Interface (Domain)
    ↓
Repository Implementation (Data)
    ↓
Firebase / Local Storage
```

Bu yapı sayesinde:
- UI, business logic'den izole edilir
- Provider'lar sadece state yönetiminden sorumludur
- Use Case'ler test edilebilir kalır

---

## 📦 Provider Tipleri ve Kullanım Alanları

### 1. **Stream Provider**
Gerçek zamanlı veri akışı için kullanılır (Firebase Firestore streams).

**Kullanım Alanları:**
- Kullanıcı profili
- Chat mesajları
- Arkadaş listesi
- Video listesi

**Örnek Yapı:**
```dart
@riverpod
Stream<List<VideoEntity>> videos(VideosRef ref) {
  final getVideosUseCase = getIt<GetVideosUseCase>();
  return getVideosUseCase.execute();
}
```

### 2. **Future Provider**
Tek seferlik async işlemler için.

**Kullanım Alanları:**
- Favori video listesi
- Kullanıcı istatistikleri
- Badge listesi

**Örnek Yapı:**
```dart
@riverpod
Future<List<BadgeEntity>> userBadges(UserBadgesRef ref) {
  final getBadgesUseCase = getIt<GetUserBadgesUseCase>();
  return getBadgesUseCase.execute();
}
```

### 3. **State Notifier Provider**
Kompleks state yönetimi ve mutasyon gerektiren durumlar için.

**Kullanım Alanları:**
- Tema yönetimi (dark/light mode)
- Quiz state
- Video oynatma kontrolü

**Örnek Yapı:**
```dart
@riverpod
class ThemeNotifier extends _$ThemeNotifier {
  @override
  ThemeMode build() {
    _loadThemeFromStorage();
    return ThemeMode.light;
  }
  
  Future<void> toggleTheme() async {
    state = state == ThemeMode.light 
        ? ThemeMode.dark 
        : ThemeMode.light;
    await _saveThemeToStorage(state);
  }
  
  Future<void> _loadThemeFromStorage() async {
    final prefs = await SharedPreferences.getInstance();
    final isDark = prefs.getBool('isDarkMode') ?? false;
    state = isDark ? ThemeMode.dark : ThemeMode.light;
  }
  
  Future<void> _saveThemeToStorage(ThemeMode mode) async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setBool('isDarkMode', mode == ThemeMode.dark);
  }
}
```

### 4. **Simple Provider**
Statik değerler veya dependency injection için.

**Kullanım Alanları:**
- GetIt servis referansları
- Sabit değerler
- Computed values

---

## 🔄 State Lifecycle ve Yönetimi

### AsyncValue Pattern

Riverpod'un **AsyncValue** yapısı, async işlemlerin 3 durumunu net şekilde yönetir:

```dart
AsyncValue<List<Video>> videosAsync = ref.watch(videosProvider);

videosAsync.when(
  data: (videos) => VideoList(videos: videos),
  loading: () => CircularProgressIndicator(),
  error: (error, stack) => ErrorWidget(error.toString()),
);
```

**Avantajları:**
- Loading state otomatik yönetilir
- Error handling merkezi ve tutarlıdır
- UI kodu temiz ve okunabilirdir

### Provider Invalidation

State güncellemeleri için:

```dart
// Manuel invalidation
ref.invalidate(videosProvider);

// Auto-refresh (dependency değiştiğinde)
@riverpod
Stream<List<Video>> favoriteVideos(FavoriteVideosRef ref) {
  final userId = ref.watch(authStateProvider).value?.uid;
  if (userId == null) return Stream.value([]);
  
  final useCase = getIt<GetFavoriteVideosUseCase>();
  return useCase.execute(userId);
}
```

---

## 🧪 Test Edilebilirlik

Riverpod, Provider'ların test edilmesini kolaylaştırır:

```dart
test('videos provider returns list of videos', () async {
  final container = ProviderContainer(
    overrides: [
      videosProvider.overrideWith((ref) {
        return Stream.value([
          VideoEntity(id: '1', word: 'Hello'),
        ]);
      }),
    ],
  );

  final videos = await container.read(videosProvider.future);
  expect(videos.length, 1);
  expect(videos.first.word, 'Hello');
});
```

---

## 📊 Provider Kategorileri

### Authentication
- `authStateProvider` - Kullanıcı kimlik durumu
- `isAuthenticatedProvider` - Boolean auth kontrolü
- `currentUserProvider` - Aktif kullanıcı bilgisi

### User Data
- `userProfileProvider` - Kullanıcı profil bilgileri
- `userStatsProvider` - Öğrenme istatistikleri
- `userBadgesProvider` - Kazanılan rozetler

### Learning
- `videosProvider` - Tüm video listesi
- `videosByCategoryProvider` - Kategoriye göre videolar
- `favoriteVideosProvider` - Favori videolar
- `dailyWordProvider` - Günün kelimesi

### Social
- `friendsProvider` - Arkadaş listesi
- `chatListProvider` - Sohbet listesi
- `chatMessagesProvider` - Belirli sohbetin mesajları

### UI State
- `themeProvider` - Tema modu (dark/light)
- `navigationProvider` - Navigasyon state

---

## ⚡ Performance Optimizasyonu

### 1. Auto-Dispose
Kullanılmayan provider'ların otomatik temizlenmesi:

```dart
@riverpod
Stream<List<Video>> videos(VideosRef ref) {
  // Auto-dispose enabled by default with @riverpod
  return getIt<GetVideosUseCase>().execute();
}
```

### 2. Select Optimization
Sadece gerekli state parçasının izlenmesi:

```dart
// ❌ Kötü: Tüm user state'ini izler
final user = ref.watch(userProvider);
final name = user.name;

// ✅ İyi: Sadece name'i izler
final name = ref.watch(userProvider.select((u) => u.name));
```

### 3. KeepAlive
Kritik provider'ların cache'de tutulması:

```dart
@Riverpod(keepAlive: true)
Stream<AuthUser?> authState(AuthStateRef ref) {
  return firebaseAuth.authStateChanges();
}
```

---

## 🔐 Dependency Injection Entegrasyonu

Riverpod ve GetIt birlikte kullanılır:

```dart
// GetIt registration (main.dart)
void setupDependencies() {
  getIt.registerLazySingleton<GetVideosUseCase>(
    () => GetVideosUseCase(getIt<VideoRepository>()),
  );
}

// Provider içinde kullanım
@riverpod
Stream<List<VideoEntity>> videos(VideosRef ref) {
  final useCase = getIt<GetVideosUseCase>();
  return useCase.execute();
}
```

**Avantajları:**
- Use Case'ler singleton olarak yönetilir
- Provider'lar sadece state yönetiminden sorumludur
- Test edilebilirlik artar

---

## 📌 Best Practices

1. **Provider'ları küçük tutun** - Her provider tek bir sorumluluğa sahip olmalı
2. **AsyncValue kullanın** - Loading ve error state'lerini otomatik yönetin
3. **Select ile optimize edin** - Gereksiz rebuild'leri engelleyin
4. **Auto-dispose varsayılan** - Özel durumlar için keepAlive kullanın
5. **Use Case'lerle iletişim** - Provider'lar doğrudan repository'lere erişmemeli

---

## 🎯 Sonuç

Silencia'da Riverpod kullanımı:
- Clean Architecture prensiplerine uyumludur
- Test edilebilir ve maintainable kod sağlar
- Firebase ile güçlü entegrasyon sunar
- Modern ve type-safe state management sağlar

Bu yapı sayesinde uygulama, ölçeklenebilir ve sürdürülebilir bir state management altyapısına sahiptir.