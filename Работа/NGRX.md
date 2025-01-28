Для начала, нужно выучить SOLID. Основной используемый принципе - Single Responsibility.

Сначала мы создаем API сервис вроде такого:
``` userapi.service.ts
import { HttpClient } from "@angular/common/http";
import { inject, Injectable } from "@angular/core";
import { Observable } from "rxjs";
import { User } from "app/pages/users-page/models/user.model";

export const API_ENDPOINTS = {
	HOST: 'http://localhost:3000/',
	USERS: 'users/'
}

@Injectable({providedIn: 'root'})
export class UserApiService {
	private http = inject(HttpClient);

	getUsers(): Observable<User[]> {
		return this.http.get<User[]>(
			API_ENDPOINTS.HOST + API_ENDPOINTS.USERS
		);
	}

	deleteUser(id: string): Observable<User> {
		return this.http.delete<User>(
			API_ENDPOINTS.HOST + API_ENDPOINTS.USERS + id
		);
	}

	addUser(user: User): Observable<User> {
		return this.http.post<User>(
			API_ENDPOINTS.HOST + API_ENDPOINTS.USERS, 
			user
		);
	}

	updateUser(id: string, user: User): Observable<User> {
		return this.http.put<User>(
			API_ENDPOINTS.HOST + API_ENDPOINTS.USERS + id, 
			user
		);
	}
}
```

Далее идет работа непосредственно с стейт-менеджером NGRX. По структуре, она напоминает Redux (таковым, по сути, он и является (NGRX - aNGular ReduX)).

Вот так должны быть организованы файлы:
```
store/
	user.actions.ts
	user.effects.ts
	user.reducer.ts
	user.selectors.ts
```

Для начала, стоит определить список действий (Action/Экшен). Как правило, экшены - это объекты, состоящие из типа (type) и сообщения (payload). Экшен, словно, контракт сигнализирует о работе с состояниями (вроде обычного CRUD).

Создать экшены (группу экшенов), можно как на примере:
```user.actions.ts
import { User } from "app/pages/users-page/models/user.model";
import { emptyProps, props } from "@ngrx/store";
import { createActionGroup } from "@ngrx/store";

export const userActions = createActionGroup({
	source: 'User',
	events: {
		loadUsers: emptyProps(),
		loadUsersSuccess: props<{ users: User[] }>(),
		loadUsersFailure: emptyProps(),
		
		updateUser: props<{ id: string, user: User }>(),
		updateUserSuccess: props<{ user: User }>(),
		updateUserFailure: emptyProps(),
		
		addUser: props<{ user: User }>(),
		addUserSuccess: props<{ user: User }>(),
		addUserFailure: emptyProps(),
		  
		
		deleteUser: props<{ id: string }>(),
		deleteUserSuccess: props<{ id: string }>(),
		deleteUserFailure: emptyProps(),
	},
});
```

Естественно, для работы с состояниями нужны состояния. Их можно выделить в одельном файле, но лучше всего их прописать в редьюсерах.

Редьюсеры - это объекты, которые возвращают нужные объекты, в обмен на экшены. Идея проста: Отправляешь экшен и, в зависимости от экшена, получаешь объект из редьюсера. Редьюсеры так же могут описать ситуацию, когда данные не пришли.

Пример редьюсера:
```user.reducers.ts
import { User } from "app/pages/users-page/models/user.model";
import { createFeature, createReducer, on } from "@ngrx/store";
import { userActions } from "./user.actions";

export interface UserState {
	isLoading: boolean;
	isError: boolean;
	data: {
		users: User[];
	}
} 

export const initialState: UserState = {
	isLoading: false,
	isError: false,
	data: {
		users: [],
	},
};

export const userFeatureKey = 'user';
export const userFeature = createFeature({
	name: userFeatureKey,
	reducer: createReducer(
		initialState,
		on(
			userActions.loadUsers,
			(state) => ({
				...state,
				isLoading: true
			})
		),
		on(
			userActions.loadUsersSuccess,
			(state, action) => ({
				...state,
				isLoading: false,
				data: { users: action.users }
			})
		),
		on(
			userActions.loadUsersFailure,
			(state) => ({
				...state,
				isLoading: false,
				isError: true
			})
		),
		on(
			userActions.updateUser,
			(state) => ({
				...state,
				isLoading: true
			})
		),
		on(
			userActions.updateUserSuccess,
			(state, action) => ({
				...state,
				isLoading: false,
				data: { 
					users: state.data.users.map(
						(user) => 
							user.id === action.user.id ? action.user : user
					) 
				}
			})
		),
		on(
			userActions.updateUserFailure,
			(state) => ({
				...state,
				isLoading: false,
				isError: true
			})
		),
		on(
			userActions.addUser,
			(state) => ({
				...state,
				isLoading: true
			})
		),
		on(
			userActions.addUserSuccess,
			(state, action) => ({
				...state,
				isLoading: false,
				data: { users: [...state.data.users, action.user] } 
			})
		),
		on(
			userActions.addUserFailure, (state) => ({
				...state,
				isLoading: false,
				isError: true
			})
		),
		on(
			userActions.deleteUser,
			(state) => ({
				...state,
				isLoading: true
			})
		),
		on(
			userActions.deleteUserSuccess,
			(state, action) => ({
				...state,
				isLoading: false,
				data: { 
					users: state.data.users.filter(
						(user) => user.id !== action.id
					) 
				}
			})
		),
		on(
			userActions.deleteUserFailure,
			(state) => ({
				...state,
				isLoading: false,
				isError: true
			})
		),
	),
});
```

Помимо редьюсера, нам необходимо создать фичу (feature) и ключ доступа. Далее, в конфигах приложения прописать провайдер нашей фичи (об этом позже).

Для чтения данных, нужно использовать селекторы. Они возвращают нужные нам данные. Пример селектора:
```user.selectors.ts
import { createFeatureSelector, createSelector } from "@ngrx/store";

import { userFeatureKey, UserState } from "./user.reducers";

const selectFeature = createFeatureSelector<UserState>(userFeatureKey);
const selectUsers = createSelector(
	selectFeature, 
	(state: UserState) => state.data.users
);
const selectIsLoading = createSelector(
	selectFeature, 
	(state: UserState) => state.isLoading
);
const selectIsError = createSelector(
	selectFeature, 
	(state: UserState) => state.isError
);

export const userSelectors = {
	selectUsers,
	selectIsLoading,
	selectIsError,
};
```

Также не стоит забывать о важном: как именно будут загружаться данные в стор? Для такого можно использовать эффекты (или SideEffects). По сути, эффект при отправлении экшена будет параллельно загружать данные в стор из API сервиса. Пример эффекта ниже:
```
import { Injectable, inject } from "@angular/core";
import { Actions, createEffect, ofType } from "@ngrx/effects";
import { catchError, map, mergeMap, switchMap } from "rxjs/operators";
import { of } from "rxjs";
import { userActions } from "./user.actions";
import { User } from "app/pages/users-page/models/user.model";
import { UserApiService } from "app/pages/users-page/services/userapi.service";

@Injectable()
export class UserEffects {
	private readonly actions$: Actions = inject(Actions);
	private readonly userApiService: UserApiService = inject(UserApiService);
  
	public loadUsers$ = createEffect(() => this.actions$.pipe(
		ofType(userActions.loadUsers),
		switchMap(() => this.userApiService.getUsers().pipe(
			map((users: User[]) => userActions.loadUsersSuccess({ users })),
			catchError(() => of(userActions.loadUsersFailure()))
		))
	));

	public updateUser$ = createEffect(() => this.actions$.pipe(
		ofType(userActions.updateUser),
		switchMap((action) => this.userApiService.updateUser(
			action.id, 
			action.user
		).pipe(
			map((user: User) => userActions.updateUserSuccess({ user })),
			catchError(() => of(userActions.updateUserFailure()))
		))
	));

	public addUser$ = createEffect(() => this.actions$.pipe(
		ofType(userActions.addUser),
		switchMap((action) => this.userApiService.addUser(action.user).pipe(
			map((user: User) => userActions.addUserSuccess({ user })),
			catchError(() => of(userActions.addUserFailure()))
		))
	));  

	public deleteUser$ = createEffect(() => this.actions$.pipe(
		ofType(userActions.deleteUser),
		switchMap((action) => this.userApiService.deleteUser(action.id).pipe(
			map(() => userActions.deleteUserSuccess({ id: action.id })),
			catchError(() => of(userActions.deleteUserFailure()))
		))
	));
}
```

Да, мы создаем потоки (каналы/observables). В нем мы привязываем экшен к эффекту через (ofType и switchMap).

Все, теперь остается работа с непосредственно компонентом. Примеры работы ниже:
```user-list.component.ts
import { Component, inject, OnInit, OnDestroy } from '@angular/core';
import { AsyncPipe, NgFor } from '@angular/common';
import { UserCardComponent } from 'app/pages/users-page/components/user-card/user-card.component';
import { userActions } from 'app/pages/users-page/store/user.actions';
import { Store } from '@ngrx/store';
import { userSelectors } from 'app/pages/users-page/store/user.selectors';
import { Subscription } from 'rxjs';

@Component({
	selector: 'app-user-list',
	imports: [AsyncPipe, UserCardComponent, NgFor],
	templateUrl: './user-list.component.html',
	styleUrl: './user-list.component.css'
})
export class UserListComponent implements OnInit, OnDestroy {
	private readonly store = inject(Store);
	
	public users$ = this.store.select(userSelectors.selectUsers);
	public isError$ = this.store.select(userSelectors.selectIsError);
	private isErrorSubscription: Subscription | undefined;

	public ngOnInit(): void {
		this.store.dispatch(userActions.loadUsers());

		this.isErrorSubscription = this.isError$.subscribe((isError) => {
			if (isError) {
				console.error('Ошибка загрузки пользователей');
			}
		});
	} 

	public ngOnDestroy(): void {
		this.isErrorSubscription?.unsubscribe();
	}
}
```

```user-list.template.ts
<div>
	<app-user-card *ngFor="let user of users$ | async" [user]="user" />
</div>
```

