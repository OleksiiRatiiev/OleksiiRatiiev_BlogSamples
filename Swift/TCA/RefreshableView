import ComposableArchitecture
import SwiftUI

@Reducer
struct RefreshableFeature : Sendable {
    @ObservableState
    struct State: Hashable, Sendable {
        var items: [String] = ["first", "second", "third"]
        var isRefreshing: Bool = false
    }

    public enum Action : Equatable, Sendable, ViewAction {
        case view(ViewAction)
        
        public enum ViewAction: Equatable, Sendable {
            case refreshFinished
            case refresh
            case cancelRefreshTapped
        }
    }
    
    private enum CancelID { case refreshRequest }
 
    @Dependency(\.continuousClock) var clock

    var body: some Reducer<State, Action> {
        Reduce { state, action in
            switch action {
            case let .view(viewAction):
                handleViewAction(&state, viewAction: viewAction)
            }
        }
    }
}

private extension RefreshableFeature {
    func handleViewAction(
        _ state: inout State,
        viewAction: Action.ViewAction
    ) -> Effect<Action> {
        switch viewAction {
        case .refreshFinished:
            state.isRefreshing = false
            return .none
        case .refresh:
            state.isRefreshing = true
            return .run { _ in
                try await clock.sleep(for: .seconds(2))
            }
            .cancellable(id: CancelID.refreshRequest)
        case .cancelRefreshTapped:
            return .cancel(id: CancelID.refreshRequest)
        }
    }
}

@ViewAction(for: RefreshableFeature.self)
struct RefreshableView: View {
    @Bindable var store: StoreOf<RefreshableFeature>

    var body: some View {
            List {
                if store.isRefreshing {
                    withAnimation(.easeIn) {
                        Button("Cancel") {
                            send(.cancelRefreshTapped)
                        }
                    }
                }
                ForEach(store.items, id: \.self) { item in
                    Text(item)
                }
            }
            .refreshable {
                defer { send(.refreshFinished) }
                await send(.refresh).finish()
            }
    }
}

#Preview {
    RefreshableView(
        store: Store(initialState: RefreshableFeature.State()) {
            RefreshableFeature()
        }
    )
}
