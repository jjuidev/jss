# React Boilerplate Code Workflow

## Step 1: Vite config

- Update **vite.config.ts** file for unified personal style code.

```ts
const viteConfig = defineConfig(({ mode }) => ({
	resolve: {
		alias: [
			{
				find: '@',
				replacement: resolve(__dirname, './src')
			}
		]
	},

	plugins: [
		react({
			babel: {
				plugins: [['babel-plugin-react-compiler']]
			}
		})
	],

	css: {
		devSourcemap: mode === 'development'
	},

	build: {
		sourcemap: mode === 'development',
		minify: mode === 'production'
	},

	server: {
		open: mode === 'development'
	}
}));

export default viteConfig;
```

## Step 2: Bootstrap

- Update **src/index.html, src/main.tsx, src/App.tsx** for unified personal style code.

```tsx
// src/index.html
<title><Project Name></title>

// src/main.tsx
const bootstrap = () => {
	const container = document.querySelector('#root')

	if (!container) {
		throw new Error("No element 'root' is defined in index.html!")
	}

	const root = createRoot(container)

	root.render(<App />)
}

void bootstrap()

// src/App.tsx
export type AppProps = {}

export const App = ({}: AppProps) => {
	return (
		<div>
			<h1><Project Name> App by JSS skill</h1>
		</div>
	)
}
```

## Step 3: Providers components (PascalCase)

- Create **src/components/Providers** files for solve `react hell providers` problem.

```tsx
// src/components/Providers/index.tsx
export type ProvidersProps = PropsWithChildren & {};

export const Providers = ({ children }: ProvidersProps) => {
	const components: ComponentWithProps[] = [
		{
			provider: StrictMode,
			props: {}
		}
	];

	const ProviderTree = buildProvidersTree(components);

	return <ProviderTree>{children}</ProviderTree>;
};

// src/components/Providers/utils/index.tsx
export type ProviderProps = PropsWithChildren & {
	[key: string]: any;
};

export type ComponentWithProps = {
	provider: ComponentType<any>;
	props?: ProviderProps;
};

export const buildProvidersTree = (componentsWithProps: ComponentWithProps[]) => {
	const initialComponent = ({ children }: { children: ReactNode }) => <>{children}</>;

	return componentsWithProps.reduce(
		(AccumulatedComponents, { provider: Provider, props: hocProps = {} }) =>
			({ children, ...props }: { children: ReactNode }) => (
				<AccumulatedComponents>
					<Provider
						{...hocProps}
						{...props}
					>
						{children}
					</Provider>
				</AccumulatedComponents>
			),
		initialComponent
	);
};
```

## Step 4: contexts (PascalCase)

- Create **src/contexts/DemoContext** files for demo purpose.

```tsx
// src/contexts/DemoContext/index.tsx
export type ExampleContextValue = {};

export const ExampleContext = createContext<ExampleContextValue | null>(null);

export type ExampleProviderProps = PropsWithChildren & {};

export const ExampleProvider = ({ children }: ExampleProviderProps) => {
	const value = {};

	return <ExampleContext.Provider value={value}>{children}</ExampleContext.Provider>;
};

// src/hooks/useExample/index.tsx
export const useExample = () => {
	const context = use(ExampleContext);

	if (!context) {
		throw new Error('useExample must be used within ExampleProvider');
	}

	return context;
};
```

## Step 5: AuthGuard hocs (PascalCase)

- Create **src/hocs/AuthGuard, src/hocs/GuestOnlyGuard** files for `react guard strategy` by `react-router-dom`

```tsx
// src/hooks/useAuth/index.ts
export const useAuth = () => {
	const isAuthenticated = true;

	return {
		isAuthenticated
	};
};

// src/hocs/AuthGuard/index.tsx
export type AuthGuardProps = PropsWithChildren & {};

export const AuthGuard = ({ children }: AuthGuardProps) => {
	const { isAuthenticated } = useAuth();

	if (!isAuthenticated) {
		return (
			<Navigate
				to="/login"
				replace
			/>
		);
	}

	return children;
};

// src/hocs/GuestOnlyGuard/index.tsx
export type GuestOnlyGuardProps = PropsWithChildren & {};

export const GuestOnlyGuard = ({ children }: GuestOnlyGuardProps) => {
	const { isAuthenticated } = useAuth();

	if (isAuthenticated) {
		return (
			<Navigate
				to="/"
				replace
			/>
		);
	}

	return children;
};
```

## Step 6: hooks (camelCase)

- Create useExample hook for demo purpose.

```tsx
// src/hooks/useExample/index.ts
export type UseExampleOptions = {
	example: string;
};

export type UseExampleResult = {
	example: string;
};

export const useExample = ({ example }: UseExampleOptions): UseExampleResult => {
	return {
		example: `${example} computed`
	};
};
```

## Step 7: layouts (PascalCase)

- Create layouts for demo purpose.

```tsx
// src/layouts/MainLayout/index.tsx
export type MainLayoutProps = PropsWithChildren & HTMLAttributes<HTMLDivElement> & {};

export const MainLayout = ({ children, ...props }: MainLayoutProps) => {
	const outlet = useOutlet();

	return (
		<div {...props}>
			<header>Header</header>
			<main>{children || outlet}</main>
			<footer>Footer</footer>
		</div>
	);
};
```

## Step 8: services (camelCase)

- Not nested folder with index file because service is single logic, not multiple logics.
- Create axios client for common service.
- Create exampleService (compatible with react-query) for demo purpose.

```tsx
// src/services/axiosClient.ts
const axiosOptions: AxiosRequestConfig = {
	baseURL: import.meta.env.VITE_API_BASE_URL,
	headers: {
		'Content-Type': 'application/json'
	}
};

export const axiosClient = axios.create(axiosOptions);
export const axiosAuthClient = axios.create(axiosOptions);

let refreshTokenRequest: Promise<string> | null = null;

const refreshTokenFunction = async () => {
	const refreshToken = useAuthStore.getState().refreshToken;

	const response = await axiosAuthClient.post('/refresh-token', { refreshToken });

	if (response.data.data) {
		const { accessToken, refreshToken } = response.data.data;

		useAuthStore.setState({
			accessToken,
			refreshToken
		});

		return accessToken;
	}

	throw new Error('Refresh token failed');
};

axiosClient.interceptors.request.use((config) => {
	const accessToken = useAuthStore.getState().accessToken;

	if (accessToken) {
		config.headers.Authorization = `Bearer ${accessToken}`;
	}

	return config;
});

axiosClient.interceptors.response.use(
	(response) => Promise.resolve(response),
	async (error) => {
		if (error.response?.status === 401 && error.response.data.message === 'accessToken jwt expired') {
			const originalRequest = error.config;

			try {
				refreshTokenRequest = refreshTokenRequest || refreshTokenFunction();
				const accessToken = await refreshTokenRequest;

				originalRequest.headers.authorization = `Bearer ${accessToken}`;
				return axiosClient(originalRequest);
			} catch (error) {
				useAuthStore.setState({
					user: null,
					accessToken: '',
					refreshToken: ''
				});

				return Promise.reject(error);
			} finally {
				refreshTokenRequest = null;
			}
		}

		return Promise.reject(error);
	}
);

// src/services/exampleService.ts
export const exampleService = {
	list: async () => {
		return axiosClient.get('/example');
	},
	detail: async (id: string) => {
		return axiosClient.get(`/example/${id}`);
	},
	create: async (data: any) => {
		return axiosClient.post('/example', data);
	},
	update: async (id: string, data: any) => {
		return axiosClient.put(`/example/${id}`, data);
	},
	delete: async (id: string) => {
		return axiosClient.delete(`/example/${id}`);
	}
};

// IMPORTANT: only update it when the user has use react-query
export const exampleQueryKeys = {
	all: ['example'] as const,
	lists: () => [...exampleQueryKeys.all, 'list'] as const,
	list: (filter: Record<string, any>) => [...exampleQueryKeys.lists(), filter] as const,
	details: () => [...exampleQueryKeys.all, 'detail'] as const,
	detail: (id: string) => [...exampleQueryKeys.details(), id] as const
};
```

## Step 9: stores (camelCase)

- Create store for demo purpose by `zustand`.

```tsx
// src/stores/useExampleStore.ts
export type ExampleState = {
	example: string;
};

export type ExampleActions = {
	setExample: (example: string) => void;
};

const initialState: ExampleState = {
	example: ''
};

export const useExampleStore = create<ExampleState & ExampleActions>((set) => ({
	...initialState,

	setExample: (example) => set({ example })
}));
```

## Step 10: utils (camelCase)

- Create utils for demo purpose.

```tsx
// src/utils/cn.ts
export function cn(...inputs: ClassValue[]) {
	return twMerge(clsx(inputs));
}
```
