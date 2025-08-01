# Navigation, State Management, ViewModel & MVVM - In-Depth Guide

## Table of Contents
1. [Navigation in Compose](#navigation-in-compose)
2. [State Management](#state-management)
3. [ViewModel Deep Dive](#viewmodel-deep-dive)
4. [MVVM Architecture](#mvvm-architecture)
5. [Advanced Patterns](#advanced-patterns)

---

## Navigation in Compose

### Basic Setup

```kotlin
// build.gradle.kts
implementation("androidx.navigation:navigation-compose:2.7.5")
implementation("androidx.hilt:hilt-navigation-compose:1.1.0")

// MainActivity.kt
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MyAppTheme {
                val navController = rememberNavController()
                AppNavigation(navController = navController)
            }
        }
    }
}
```

### Navigation Graph Setup

```kotlin
@Composable
fun AppNavigation(navController: NavHostController) {
    NavHost(
        navController = navController,
        startDestination = Screen.Home.route
    ) {
        composable(Screen.Home.route) {
            HomeScreen(
                onNavigateToDetail = { id ->
                    navController.navigate(Screen.Detail.createRoute(id))
                },
                onNavigateToProfile = {
                    navController.navigate(Screen.Profile.route)
                }
            )
        }
        
        composable(
            route = Screen.Detail.route,
            arguments = listOf(
                navArgument("itemId") {
                    type = NavType.StringType
                    defaultValue = ""
                }
            )
        ) { backStackEntry ->
            val itemId = backStackEntry.arguments?.getString("itemId") ?: ""
            DetailScreen(
                itemId = itemId,
                onNavigateBack = {
                    navController.popBackStack()
                },
                onNavigateToEdit = { id ->
                    navController.navigate(Screen.Edit.createRoute(id))
                }
            )
        }
        
        composable(Screen.Profile.route) {
            ProfileScreen(
                onNavigateBack = {
                    navController.popBackStack()
                }
            )
        }
        
        composable(
            route = Screen.Edit.route,
            arguments = listOf(
                navArgument("itemId") { type = NavType.StringType }
            )
        ) { backStackEntry ->
            val itemId = backStackEntry.arguments?.getString("itemId") ?: ""
            EditScreen(
                itemId = itemId,
                onSaveComplete = {
                    // Navigate back to detail with result
                    navController.previousBackStackEntry
                        ?.savedStateHandle
                        ?.set("item_updated", true)
                    navController.popBackStack()
                }
            )
        }
    }
}
```

### Screen Definitions (Type-Safe Navigation)

```kotlin
sealed class Screen(val route: String) {
    object Home : Screen("home")
    object Profile : Screen("profile")
    
    object Detail : Screen("detail/{itemId}") {
        fun createRoute(itemId: String) = "detail/$itemId"
    }
    
    object Edit : Screen("edit/{itemId}") {
        fun createRoute(itemId: String) = "edit/$itemId"
    }
    
    // Complex route with multiple parameters
    object Search : Screen("search?query={query}&category={category}") {
        fun createRoute(query: String = "", category: String = "") = 
            "search?query=$query&category=$category"
    }
}
```

### Advanced Navigation Patterns

#### Bottom Navigation
```kotlin
@Composable
fun MainScreen() {
    val navController = rememberNavController()
    val navBackStackEntry by navController.currentBackStackEntryAsState()
    val currentDestination = navBackStackEntry?.destination
    
    Scaffold(
        bottomBar = {
            NavigationBar {
                bottomNavItems.forEach { screen ->
                    NavigationBarItem(
                        icon = { Icon(screen.icon, contentDescription = null) },
                        label = { Text(screen.label) },
                        selected = currentDestination?.hierarchy?.any { 
                            it.route == screen.route 
                        } == true,
                        onClick = {
                            navController.navigate(screen.route) {
                                popUpTo(navController.graph.findStartDestination().id) {
                                    saveState = true
                                }
                                launchSingleTop = true
                                restoreState = true
                            }
                        }
                    )
                }
            }
        }
    ) { innerPadding ->
        NavHost(
            navController = navController,
            startDestination = BottomNavScreen.Home.route,
            modifier = Modifier.padding(innerPadding)
        ) {
            // Navigation graph
        }
    }
}
```

#### Nested Navigation
```kotlin
fun NavGraphBuilder.authNavGraph(navController: NavHostController) {
    navigation(
        startDestination = AuthScreen.Login.route,
        route = "auth"
    ) {
        composable(AuthScreen.Login.route) {
            LoginScreen(
                onLoginSuccess = {
                    navController.navigate("main") {
                        popUpTo("auth") { inclusive = true }
                    }
                },
                onNavigateToRegister = {
                    navController.navigate(AuthScreen.Register.route)
                }
            )
        }
        
        composable(AuthScreen.Register.route) {
            RegisterScreen(
                onRegisterSuccess = {
                    navController.navigate("main") {
                        popUpTo("auth") { inclusive = true }
                    }
                },
                onNavigateBack = {
                    navController.popBackStack()
                }
            )
        }
    }
}
```

#### Navigation with Results
```kotlin
// Sending result back
@Composable
fun EditScreen(
    onSaveComplete: (String) -> Unit
) {
    // ... screen content
    
    Button(
        onClick = {
            // Save logic
            navController.previousBackStackEntry
                ?.savedStateHandle
                ?.set("edit_result", EditResult(id = itemId, success = true))
            navController.popBackStack()
        }
    ) {
        Text("Save")
    }
}

// Receiving result
@Composable
fun DetailScreen(navController: NavHostController) {
    val savedStateHandle = navController.currentBackStackEntry?.savedStateHandle
    
    LaunchedEffect(savedStateHandle) {
        savedStateHandle?.let { handle ->
            handle.getLiveData<EditResult>("edit_result").observe(lifecycleOwner) { result ->
                if (result != null) {
                    // Handle the result
                    showSnackbar("Item updated successfully")
                    handle.remove<EditResult>("edit_result")
                }
            }
        }
    }
}
```

---

## State Management

### Local State with `remember`

```kotlin
@Composable
fun CounterScreen() {
    // Simple state
    var count by remember { mutableStateOf(0) }
    
    // State with key (recomposes when key changes)
    var userData by remember(userId) { 
        mutableStateOf(loadUserData(userId)) 
    }
    
    // Complex state object
    var formState by remember {
        mutableStateOf(
            FormState(
                name = "",
                email = "",
                isValid = false
            )
        )
    }
    
    Column {
        Text("Count: $count")
        Button(onClick = { count++ }) {
            Text("Increment")
        }
        
        TextField(
            value = formState.name,
            onValueChange = { newName ->
                formState = formState.copy(
                    name = newName,
                    isValid = validateForm(newName, formState.email)
                )
            },
            label = { Text("Name") }
        )
    }
}

data class FormState(
    val name: String,
    val email: String,
    val isValid: Boolean
)
```

### `rememberSaveable` for Configuration Changes

```kotlin
@Composable
fun SearchScreen() {
    // Survives configuration changes
    var searchQuery by rememberSaveable { mutableStateOf("") }
    var selectedFilter by rememberSaveable { mutableStateOf(FilterType.ALL) }
    
    // Custom saver for complex objects
    var searchState by rememberSaveable(
        saver = SearchState.Saver
    ) {
        mutableStateOf(SearchState())
    }
    
    // ... UI implementation
}

@Parcelize
data class SearchState(
    val query: String = "",
    val filters: List<String> = emptyList(),
    val sortOrder: SortOrder = SortOrder.NEWEST
) : Parcelable {
    companion object {
        val Saver: Saver<MutableState<SearchState>, *> = Saver(
            save = { it.value },
            restore = { mutableStateOf(it) }
        )
    }
}
```

### State Hoisting Patterns

```kotlin
// Stateful composable (holds state)
@Composable
fun StatefulCounter() {
    var count by remember { mutableStateOf(0) }
    
    StatelessCounter(
        count = count,
        onIncrement = { count++ },
        onDecrement = { count-- }
    )
}

// Stateless composable (pure function)
@Composable
fun StatelessCounter(
    count: Int,
    onIncrement: () -> Unit,
    onDecrement: () -> Unit,
    modifier: Modifier = Modifier
) {
    Row(
        modifier = modifier,
        horizontalArrangement = Arrangement.spacedBy(16.dp),
        verticalAlignment = Alignment.CenterVertically
    ) {
        Button(onClick = onDecrement) {
            Text("-")
        }
        Text(
            text = count.toString(),
            style = MaterialTheme.typography.headlineMedium
        )
        Button(onClick = onIncrement) {
            Text("+")
        }
    }
}
```

### `derivedStateOf` for Computed Values

```kotlin
@Composable
fun ExpensiveCalculationScreen() {
    val items by viewModel.items.collectAsState()
    val filter by viewModel.filter.collectAsState()
    
    // Only recomputes when items or filter changes
    val filteredItems by remember {
        derivedStateOf {
            items.filter { item ->
                when (filter) {
                    FilterType.ACTIVE -> item.isActive
                    FilterType.COMPLETED -> !item.isActive
                    FilterType.ALL -> true
                }
            }
        }
    }
    
    // Expensive computation
    val statistics by remember {
        derivedStateOf {
            filteredItems.let { items ->
                Statistics(
                    total = items.size,
                    completed = items.count { !it.isActive },
                    averageScore = items.map { it.score }.average()
                )
            }
        }
    }
    
    LazyColumn {
        item {
            StatisticsCard(statistics = statistics)
        }
        items(filteredItems) { item ->
            ItemCard(item = item)
        }
    }
}
```

### Side Effects Management

```kotlin
@Composable
fun UserProfileScreen(userId: String) {
    var userProfile by remember { mutableStateOf<UserProfile?>(null) }
    var isLoading by remember { mutableStateOf(false) }
    
    // LaunchedEffect - runs when composition starts or key changes
    LaunchedEffect(userId) {
        isLoading = true
        try {
            userProfile = userRepository.getUser(userId)
        } catch (e: Exception) {
            // Handle error
        } finally {
            isLoading = false
        }
    }
    
    // DisposableEffect - cleanup when composable leaves composition
    DisposableEffect(Unit) {
        val listener = createLocationListener()
        locationManager.addListener(listener)
        
        onDispose {
            locationManager.removeListener(listener)
        }
    }
    
    // SideEffect - runs after every successful recomposition
    SideEffect {
        analytics.trackScreenView("UserProfile", userId)
    }
    
    if (isLoading) {
        LoadingIndicator()
    } else {
        userProfile?.let { profile ->
            ProfileContent(profile = profile)
        }
    }
}
```

---

## ViewModel Deep Dive

### Basic ViewModel Structure

```kotlin
class HomeViewModel @Inject constructor(
    private val repository: HomeRepository,
    private val analytics: AnalyticsService
) : ViewModel() {
    
    // Private mutable state
    private val _uiState = MutableStateFlow(HomeUiState())
    val uiState: StateFlow<HomeUiState> = _uiState.asStateFlow()
    
    // Alternative: Using MutableLiveData
    private val _events = MutableLiveData<HomeEvent>()
    val events: LiveData<HomeEvent> = _events
    
    init {
        loadInitialData()
    }
    
    fun loadInitialData() {
        viewModelScope.launch {
            _uiState.update { it.copy(isLoading = true) }
            
            try {
                val data = repository.getHomeData()
                _uiState.update { currentState ->
                    currentState.copy(
                        data = data,
                        isLoading = false,
                        error = null
                    )
                }
            } catch (exception: Exception) {
                _uiState.update { currentState ->
                    currentState.copy(
                        isLoading = false,
                        error = exception.message
                    )
                }
            }
        }
    }
    
    fun refreshData() {
        analytics.track("home_refresh")
        loadInitialData()
    }
    
    fun onItemClick(itemId: String) {
        viewModelScope.launch {
            _events.value = HomeEvent.NavigateToDetail(itemId)
        }
    }
    
    override fun onCleared() {
        super.onCleared()
        // Cleanup resources
    }
}

data class HomeUiState(
    val data: List<HomeItem> = emptyList(),
    val isLoading: Boolean = false,
    val error: String? = null,
    val selectedFilter: FilterType = FilterType.ALL
)

sealed class HomeEvent {
    data class NavigateToDetail(val itemId: String) : HomeEvent()
    object ShowError : HomeEvent()
    data class ShowSnackbar(val message: String) : HomeEvent()
}
```

### Advanced ViewModel Patterns

#### Combining Multiple Data Sources
```kotlin
class DashboardViewModel @Inject constructor(
    private val userRepository: UserRepository,
    private val notificationRepository: NotificationRepository,
    private val weatherRepository: WeatherRepository
) : ViewModel() {
    
    private val _uiState = MutableStateFlow(DashboardUiState())
    val uiState: StateFlow<DashboardUiState> = _uiState.asStateFlow()
    
    init {
        // Combine multiple flows
        viewModelScope.launch {
            combine(
                userRepository.getCurrentUser(),
                notificationRepository.getUnreadCount(),
                weatherRepository.getCurrentWeather()
            ) { user, notificationCount, weather ->
                DashboardData(
                    user = user,
                    unreadNotifications = notificationCount,
                    weather = weather
                )
            }.catch { exception ->
                _uiState.update { it.copy(error = exception.message) }
            }.collect { dashboardData ->
                _uiState.update { currentState ->
                    currentState.copy(
                        data = dashboardData,
                        isLoading = false
                    )
                }
            }
        }
    }
}
```

#### Handling User Actions with Sealed Classes
```kotlin
class TaskViewModel @Inject constructor(
    private val taskRepository: TaskRepository
) : ViewModel() {
    
    private val _uiState = MutableStateFlow(TaskUiState())
    val uiState: StateFlow<TaskUiState> = _uiState.asStateFlow()
    
    fun handleAction(action: TaskAction) {
        when (action) {
            is TaskAction.LoadTasks -> loadTasks(action.filter)
            is TaskAction.AddTask -> addTask(action.task)
            is TaskAction.UpdateTask -> updateTask(action.task)
            is TaskAction.DeleteTask -> deleteTask(action.taskId)
            is TaskAction.ToggleComplete -> toggleTaskComplete(action.taskId)
        }
    }
    
    private fun addTask(task: Task) {
        viewModelScope.launch {
            try {
                val newTask = taskRepository.addTask(task)
                _uiState.update { currentState ->
                    currentState.copy(
                        tasks = currentState.tasks + newTask
                    )
                }
            } catch (exception: Exception) {
                // Handle error
            }
        }
    }
    
    private fun updateTask(task: Task) {
        viewModelScope.launch {
            try {
                val updatedTask = taskRepository.updateTask(task)
                _uiState.update { currentState ->
                    currentState.copy(
                        tasks = currentState.tasks.map { 
                            if (it.id == updatedTask.id) updatedTask else it 
                        }
                    )
                }
            } catch (exception: Exception) {
                // Handle error
            }
        }
    }
}

sealed class TaskAction {
    data class LoadTasks(val filter: TaskFilter) : TaskAction()
    data class AddTask(val task: Task) : TaskAction()
    data class UpdateTask(val task: Task) : TaskAction()
    data class DeleteTask(val taskId: String) : TaskAction()
    data class ToggleComplete(val taskId: String) : TaskAction()
}
```

#### ViewModel with Paging
```kotlin
class PostListViewModel @Inject constructor(
    private val postRepository: PostRepository
) : ViewModel() {
    
    val posts: Flow<PagingData<Post>> = Pager(
        config = PagingConfig(
            pageSize = 20,
            enablePlaceholders = false
        ),
        pagingSourceFactory = { postRepository.getPostsPagingSource() }
    ).flow.cachedIn(viewModelScope)
    
    private val _searchQuery = MutableStateFlow("")
    val searchQuery: StateFlow<String> = _searchQuery.asStateFlow()
    
    val searchResults: Flow<PagingData<Post>> = searchQuery
        .debounce(300)
        .distinctUntilChanged()
        .flatMapLatest { query ->
            if (query.isBlank()) {
                flowOf(PagingData.empty())
            } else {
                Pager(
                    config = PagingConfig(pageSize = 20),
                    pagingSourceFactory = { postRepository.searchPosts(query) }
                ).flow
            }
        }.cachedIn(viewModelScope)
    
    fun updateSearchQuery(query: String) {
        _searchQuery.value = query
    }
}
```

---

## MVVM Architecture

### Complete MVVM Structure

#### Model Layer
```kotlin
// Data classes
data class User(
    val id: String,
    val name: String,
    val email: String,
    val avatar: String?
)

data class Post(
    val id: String,
    val title: String,
    val content: String,
    val authorId: String,
    val createdAt: LocalDateTime,
    val likes: Int
)

// Repository Interface
interface UserRepository {
    suspend fun getUser(id: String): User
    suspend fun updateUser(user: User): User
    fun observeUser(id: String): Flow<User>
}

// Repository Implementation
@Singleton
class UserRepositoryImpl @Inject constructor(
    private val apiService: ApiService,
    private val localDataSource: UserLocalDataSource,
    private val networkManager: NetworkManager
) : UserRepository {
    
    override suspend fun getUser(id: String): User {
        return if (networkManager.isConnected()) {
            try {
                val userDto = apiService.getUser(id)
                val user = userDto.toDomain()
                localDataSource.saveUser(user)
                user
            } catch (exception: Exception) {
                localDataSource.getUser(id) ?: throw exception
            }
        } else {
            localDataSource.getUser(id) ?: throw NetworkException("No internet connection")
        }
    }
    
    override fun observeUser(id: String): Flow<User> {
        return localDataSource.observeUser(id)
            .combine(networkManager.networkState) { localUser, isConnected ->
                if (isConnected && localUser != null) {
                    // Sync with server in background
                    syncUserInBackground(id)
                }
                localUser
            }
            .filterNotNull()
    }
}

// Use Cases (Optional layer for complex business logic)
class GetUserUseCase @Inject constructor(
    private val userRepository: UserRepository
) {
    suspend operator fun invoke(userId: String): Result<User> {
        return try {
            val user = userRepository.getUser(userId)
            Result.success(user)
        } catch (exception: Exception) {
            Result.failure(exception)
        }
    }
}
```

#### ViewModel Layer
```kotlin
@HiltViewModel
class UserProfileViewModel @Inject constructor(
    private val getUserUseCase: GetUserUseCase,
    private val updateUserUseCase: UpdateUserUseCase,
    private val savedStateHandle: SavedStateHandle
) : ViewModel() {
    
    private val userId: String = savedStateHandle.get<String>("userId") ?: ""
    
    private val _uiState = MutableStateFlow(UserProfileUiState())
    val uiState: StateFlow<UserProfileUiState> = _uiState.asStateFlow()
    
    private val _uiEvent = Channel<UserProfileUiEvent>()
    val uiEvent: Flow<UserProfileUiEvent> = _uiEvent.receiveAsFlow()
    
    init {
        loadUser()
    }
    
    fun onAction(action: UserProfileAction) {
        when (action) {
            is UserProfileAction.LoadUser -> loadUser()
            is UserProfileAction.UpdateProfile -> updateProfile(action.user)
            is UserProfileAction.RefreshData -> refreshData()
            is UserProfileAction.RetryLoad -> retryLoad()
        }
    }
    
    private fun loadUser() {
        viewModelScope.launch {
            _uiState.update { it.copy(isLoading = true, error = null) }
            
            getUserUseCase(userId)
                .onSuccess { user ->
                    _uiState.update { 
                        it.copy(
                            user = user,
                            isLoading = false
                        )
                    }
                }
                .onFailure { exception ->
                    _uiState.update { 
                        it.copy(
                            isLoading = false,
                            error = exception.message
                        )
                    }
                }
        }
    }
    
    private fun updateProfile(user: User) {
        viewModelScope.launch {
            _uiState.update { it.copy(isUpdating = true) }
            
            updateUserUseCase(user)
                .onSuccess { updatedUser ->
                    _uiState.update { 
                        it.copy(
                            user = updatedUser,
                            isUpdating = false
                        )
                    }
                    _uiEvent.send(UserProfileUiEvent.ShowSuccess("Profile updated"))
                }
                .onFailure { exception ->
                    _uiState.update { it.copy(isUpdating = false) }
                    _uiEvent.send(UserProfileUiEvent.ShowError(exception.message ?: "Update failed"))
                }
        }
    }
}

// UI State
data class UserProfileUiState(
    val user: User? = null,
    val isLoading: Boolean = false,
    val isUpdating: Boolean = false,
    val error: String? = null
)

// UI Actions
sealed class UserProfileAction {
    object LoadUser : UserProfileAction()
    data class UpdateProfile(val user: User) : UserProfileAction()
    object RefreshData : UserProfileAction()
    object RetryLoad : UserProfileAction()
}

// UI Events
sealed class UserProfileUiEvent {
    data class ShowSuccess(val message: String) : UserProfileUiEvent()
    data class ShowError(val message: String) : UserProfileUiEvent()
    object NavigateBack : UserProfileUiEvent()
}
```

#### View Layer (Compose)
```kotlin
@Composable
fun UserProfileScreen(
    viewModel: UserProfileViewModel = hiltViewModel(),
    onNavigateBack: () -> Unit
) {
    val uiState by viewModel.uiState.collectAsState()
    val context = LocalContext.current
    
    // Handle UI events
    LaunchedEffect(Unit) {
        viewModel.uiEvent.collect { event ->
            when (event) {
                is UserProfileUiEvent.ShowSuccess -> {
                    Toast.makeText(context, event.message, Toast.LENGTH_SHORT).show()
                }
                is UserProfileUiEvent.ShowError -> {
                    Toast.makeText(context, event.message, Toast.LENGTH_LONG).show()
                }
                is UserProfileUiEvent.NavigateBack -> {
                    onNavigateBack()
                }
            }
        }
    }
    
    UserProfileContent(
        uiState = uiState,
        onAction = viewModel::onAction,
        onNavigateBack = onNavigateBack
    )
}

@Composable
private fun UserProfileContent(
    uiState: UserProfileUiState,
    onAction: (UserProfileAction) -> Unit,
    onNavigateBack: () -> Unit
) {
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp)
    ) {
        TopAppBar(
            title = { Text("Profile") },
            navigationIcon = {
                IconButton(onClick = onNavigateBack) {
                    Icon(Icons.Default.ArrowBack, contentDescription = "Back")
                }
            }
        )
        
        when {
            uiState.isLoading -> {
                Box(
                    modifier = Modifier.fillMaxSize(),
                    contentAlignment = Alignment.Center
                ) {
                    CircularProgressIndicator()
                }
            }
            
            uiState.error != null -> {
                ErrorContent(
                    error = uiState.error,
                    onRetry = { onAction(UserProfileAction.RetryLoad) }
                )
            }
            
            uiState.user != null -> {
                ProfileForm(
                    user = uiState.user,
                    isUpdating = uiState.isUpdating,
                    onUpdateProfile = { user ->
                        onAction(UserProfileAction.UpdateProfile(user))
                    }
                )
            }
        }
    }
}
```

---

## Advanced Patterns

### State Management with Multiple ViewModels

```kotlin
// Shared ViewModel for app-wide state
@Singleton
class AppStateViewModel @Inject constructor(
    private val userRepository: UserRepository,
    private val settingsRepository: SettingsRepository
) : ViewModel() {
    
    private val _appState = MutableStateFlow(AppState())
    val appState: StateFlow<AppState> = _appState.asStateFlow()
    
    val isLoggedIn: StateFlow<Boolean> = appState
        .map { it.currentUser != null }
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            initialValue = false
        )
    
    fun login(user: User) {
        _appState.update { it.copy(currentUser = user) }
    }
    
    fun logout() {
        viewModelScope.launch {
            userRepository.clearUserData()
            _appState.update { AppState() }
        }
    }
}

// Usage in Activity/Application level
@Composable
fun App() {
    val appStateViewModel: AppStateViewModel = hiltViewModel()
    val isLoggedIn by appStateViewModel.isLoggedIn.collectAsState()
    
    if (isLoggedIn) {
        MainNavigation()
    } else {
        AuthNavigation()
    }
}
```

### Repository Pattern with Caching

```kotlin
@Singleton
class PostRepositoryImpl @Inject constructor(
    private val apiService: ApiService,
    private val postDao: PostDao,
    private val networkManager: NetworkManager
) : PostRepository {
    
    private val memoryCache = LruCache<String, Post>(maxSize = 50)
    
    override fun getPosts(): Flow<List<Post>> = flow {
        // Emit cached data first
        val cachedPosts = postDao.getAllPosts()
        emit(cachedPosts)
        
        // Fetch fresh data if connected
        if (networkManager.isConnected()) {
            try {
                val freshPosts = apiService.getPosts()
                postDao.insertPosts(freshPosts)
                
                // Update memory cache
                freshPosts.forEach { post ->
                    memoryCache.put(post.id, post)
                }
                
                emit(freshPosts)
            } catch (exception: Exception) {
                // Network error - stick with cached data
                Timber.w(exception, "Failed to fetch fresh posts")
            }
        }
    }.flowOn(Dispatchers.IO)
    
    override suspend fun getPost(id: String): Post {
        // Check memory cache first
        memoryCache.get(id)?.let { return it }
        
        // Check local database
        postDao.getPost(id)?.let { post ->
            memoryCache.put(id, post)
            return post
        }
        
        // Fetch from network
        if (networkManager.isConnected()) {
            val post = apiService.getPost(id)
            postDao.insertPost(post)
            memoryCache.put(id, post)
            return post
        }
        
        throw NetworkException("Post not found and no internet connection")
    }
}
```

### Error Handling Strategy

```kotlin
// Custom Result wrapper
sealed class Resource<out T> {
    data class Success<T>(val data: T) : Resource<T>()
    data class Error(val exception: Throwable) : Resource<Nothing>()
    object Loading : Resource<Nothing>()
}

// Extension functions
fun <T> Resource<T>.onSuccess(action: (T) -> Unit): Resource<T> {
    if (this is Resource.Success) action(data)
    return this
}

fun <T> Resource<T>.onError(action: (Throwable) -> Unit): Resource<T> {
    if (this is Resource.Error) action(exception)
    return this
}

// ViewModel with Resource
class ProductViewModel @Inject constructor(
    private val productRepository: ProductRepository
) : ViewModel() {
    
    private val _products = MutableStateFlow<Resource<List<Product>>>(Resource.Loading)
    val products: StateFlow<Resource<List<Product>>> = _products.asStateFlow()
    
    fun loadProducts() {
        viewModelScope.launch {
            _products.value = Resource.Loading
            
            try {
                val productList = productRepository.getProducts()
                _products.value = Resource.Success(productList)
            } catch (exception: Exception) {
                _products.value = Resource.Error(exception)
            }
        }
    }
}

// Usage in Composable
@Composable
fun ProductListScreen(
    viewModel: ProductViewModel = hiltViewModel()
) {
    val productsResource by viewModel.products.collectAsState()
    
    when (productsResource) {
        is Resource.Loading -> {
            LoadingIndicator()
        }
        is Resource.Success -> {
            ProductList(products = productsResource.data)
        }
        is Resource.Error -> {
            ErrorMessage(
                error = productsResource.exception,
                onRetry = { viewModel.loadProducts() }
            )
        }
    }
}
```

### Testing Strategies

#### ViewModel Testing
```kotlin
@ExperimentalCoroutinesTest
class HomeViewModelTest {
    
    @get:Rule
    val mainDispatcherRule = MainDispatcherRule()
    
    private val mockRepository = mockk<HomeRepository>()
    private lateinit var viewModel: HomeViewModel
    
    @Before
    fun setup() {
        viewModel = HomeViewModel(mockRepository)
    }
    
    @Test
    fun `loadData should update state to loading then success`() = runTest {
        // Given
        val mockData = listOf(HomeItem("1", "Test"))
        coEvery { mockRepository.getHomeData() } returns mockData
        
        // When
        viewModel.loadData()
        
        // Then
        val states = mutableListOf<HomeUiState>()
        val job = launch {
            viewModel.uiState.collect { states.add(it) }
        }
        
        advanceUntilIdle()
        
        assertTrue(states.any { it.isLoading })
        assertTrue(states.last().data == mockData)
        assertFalse(states.last().isLoading)
        
        job.cancel()
    }
    
    @Test
    fun `loadData should handle error correctly`() = runTest {
        // Given
        val errorMessage = "Network error"
        coEvery { mockRepository.getHomeData() } throws Exception(errorMessage)
        
        // When
        viewModel.loadData()
        advanceUntilIdle()
        
        // Then
        val currentState = viewModel.uiState.value
        assertFalse(currentState.isLoading)
        assertEquals(errorMessage, currentState.error)
    }
}
```

#### Repository Testing
```kotlin
@ExperimentalCoroutinesTest
class UserRepositoryTest {
    
    private val mockApiService = mockk<ApiService>()
    private val mockLocalDataSource = mockk<UserLocalDataSource>()
    private val mockNetworkManager = mockk<NetworkManager>()
    
    private lateinit var repository: UserRepositoryImpl
    
    @Before
    fun setup() {
        repository = UserRepositoryImpl(
            mockApiService,
            mockLocalDataSource,
            mockNetworkManager
        )
    }
    
    @Test
    fun `getUser should return cached data when offline`() = runTest {
        // Given
        val userId = "123"
        val cachedUser = User(userId, "John", "john@example.com", null)
        
        every { mockNetworkManager.isConnected() } returns false
        coEvery { mockLocalDataSource.getUser(userId) } returns cachedUser
        
        // When
        val result = repository.getUser(userId)
        
        // Then
        assertEquals(cachedUser, result)
        coVerify { mockLocalDataSource.getUser(userId) }
        coVerify(exactly = 0) { mockApiService.getUser(any()) }
    }
}
```

### Performance Optimization Patterns

#### Lazy Loading with Pagination
```kotlin
@HiltViewModel
class FeedViewModel @Inject constructor(
    private val feedRepository: FeedRepository
) : ViewModel() {
    
    private val _feedState = MutableStateFlow(FeedState())
    val feedState: StateFlow<FeedState> = _feedState.asStateFlow()
    
    private var currentPage = 0
    private val pageSize = 20
    
    init {
        loadInitialFeed()
    }
    
    fun loadInitialFeed() {
        viewModelScope.launch {
            _feedState.update { it.copy(isLoading = true, error = null) }
            
            try {
                currentPage = 0
                val posts = feedRepository.getFeedPosts(page = 0, pageSize = pageSize)
                _feedState.update { 
                    it.copy(
                        posts = posts,
                        isLoading = false,
                        hasMorePages = posts.size == pageSize
                    )
                }
            } catch (exception: Exception) {
                _feedState.update { 
                    it.copy(
                        isLoading = false,
                        error = exception.message
                    )
                }
            }
        }
    }
    
    fun loadMorePosts() {
        val currentState = _feedState.value
        if (currentState.isLoadingMore || !currentState.hasMorePages) return
        
        viewModelScope.launch {
            _feedState.update { it.copy(isLoadingMore = true) }
            
            try {
                currentPage++
                val newPosts = feedRepository.getFeedPosts(currentPage, pageSize)
                _feedState.update { currentState ->
                    currentState.copy(
                        posts = currentState.posts + newPosts,
                        isLoadingMore = false,
                        hasMorePages = newPosts.size == pageSize
                    )
                }
            } catch (exception: Exception) {
                currentPage-- // Revert page increment
                _feedState.update { 
                    it.copy(
                        isLoadingMore = false,
                        error = exception.message
                    )
                }
            }
        }
    }
}

data class FeedState(
    val posts: List<Post> = emptyList(),
    val isLoading: Boolean = false,
    val isLoadingMore: Boolean = false,
    val hasMorePages: Boolean = true,
    val error: String? = null
)

// Usage in Compose
@Composable
fun FeedScreen(viewModel: FeedViewModel = hiltViewModel()) {
    val feedState by viewModel.feedState.collectAsState()
    val listState = rememberLazyListState()
    
    // Trigger load more when near end
    LaunchedEffect(listState) {
        snapshotFlow { listState.layoutInfo.visibleItemsInfo }
            .collect { visibleItems ->
                val lastIndex = feedState.posts.size - 1
                val shouldLoadMore = visibleItems.any { it.index >= lastIndex - 3 }
                
                if (shouldLoadMore && feedState.hasMorePages && !feedState.isLoadingMore) {
                    viewModel.loadMorePosts()
                }
            }
    }
    
    LazyColumn(state = listState) {
        items(feedState.posts) { post ->
            PostCard(post = post)
        }
        
        if (feedState.isLoadingMore) {
            item {
                LoadingIndicator(
                    modifier = Modifier
                        .fillMaxWidth()
                        .padding(16.dp)
                )
            }
        }
    }
}
```

#### Memory Management
```kotlin
class ImageCacheManager @Inject constructor() {
    private val memoryCache: LruCache<String, Bitmap> = object : LruCache<String, Bitmap>(maxSize) {
        override fun sizeOf(key: String, bitmap: Bitmap): Int {
            return bitmap.byteCount / 1024
        }
        
        override fun entryRemoved(
            evicted: Boolean,
            key: String,
            oldValue: Bitmap,
            newValue: Bitmap?
        ) {
            if (!oldValue.isRecycled) {
                oldValue.recycle()
            }
        }
    }
    
    private val maxSize: Int
        get() {
            val maxMemory = (Runtime.getRuntime().maxMemory() / 1024).toInt()
            return maxMemory / 8 // Use 1/8th of available memory
        }
    
    fun getBitmap(key: String): Bitmap? = memoryCache.get(key)
    
    fun putBitmap(key: String, bitmap: Bitmap) {
        if (getBitmap(key) == null) {
            memoryCache.put(key, bitmap)
        }
    }
}
```

### Best Practices Summary

#### State Management
- Use `StateFlow` for UI state in ViewModels
- Prefer `derivedStateOf` for computed values
- Implement proper state hoisting
- Use appropriate side effect APIs
- Consider using `rememberSaveable` for configuration changes

#### Navigation
- Implement type-safe navigation with sealed classes
- Use proper navigation options for different scenarios
- Handle navigation results appropriately
- Implement proper back stack management

#### ViewModel
- Keep ViewModels UI-agnostic
- Use coroutines for asynchronous operations
- Implement proper error handling
- Follow single responsibility principle
- Use dependency injection

#### MVVM Architecture
- Separate concerns clearly (View, ViewModel, Model)
- Use repository pattern for data management
- Implement proper caching strategies
- Use Use Cases for complex business logic
- Follow clean architecture principles

#### Testing
- Write unit tests for ViewModels and Repositories
- Use test doubles (mocks, fakes) appropriately
- Test different scenarios (success, error, loading)
- Use proper test coroutines for async code

This comprehensive guide covers the essential patterns and practices for building robust Android applications with Jetpack Compose, proper state management, and MVVM architecture. Each pattern is demonstrated with practical examples that you can adapt to your specific use cases.