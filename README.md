from kivy.app import App
from kivy.graphics import Color, Rectangle
from kivy.uix.floatlayout import FloatLayout
from kivy.uix.button import Button
from kivy.core.window import Window

class Player(FloatLayout):
    def __init__(self, player_id, **kwargs):
        super(Player, self).__init__(**kwargs)
        self.size = (50, 50)
        self.pos = (Window.width // 4 if player_id == 1 else Window.width * 3 // 4, Window.height // 2)
        self.frozen = False
        self.player_id = player_id
        self.bind(pos=self.update_rect)

        # Cria um botão para congelar/descongelar apenas para o jogador ativo
        if self.player_id == 1:  # Apenas o jogador 1 vê o botão
            self.freeze_button = Button(text='Ativar Freezer', size_hint=(None, None), size=(150, 50), pos=(10, 10))
            self.freeze_button.bind(on_press=self.toggle_freeze)
            self.add_widget(self.freeze_button)

    def update_rect(self, *args):
        self.canvas.clear()
        with self.canvas:
            Color(0, 0, 1, 1 if self.frozen else 1)  # Azul se congelado
            Rectangle(pos=self.pos, size=self.size)

    def move(self, dx, dy):
        if not self.frozen:
            self.pos = (self.x + dx, self.y + dy)

    def toggle_freeze(self, *args):
        self.frozen = not self.frozen
        if self.player_id == 1:  # Se o jogador 1 congela, congela o jogador 2
            app.player2.frozen = self.frozen
            app.player2.update_rect()

class GameApp(App):
    def build(self):
        layout = FloatLayout()
        global app  # Para acesso global ao app
        app = self
        
        # Criação de dois jogadores
        self.player1 = Player(player_id=1)
        self.player2 = Player(player_id=2)
        
        layout.add_widget(self.player1)
        layout.add_widget(self.player2)

        # Configurando eventos de toque
        Window.bind(on_touch_move=self.move_player)
        return layout

    def move_player(self, window, touch):
        # Permite movimentação apenas se o jogador não estiver congelado
        if not app.player1.frozen:
            dx = touch.x - app.player1.x - app.player1.width / 2
            dy = touch.y - app.player1.y - app.player1.height / 2
            app.player1.move(dx, dy)

if __name__ == '__main__':
    GameApp().run()
    
