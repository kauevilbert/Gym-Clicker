import tkinter as tk
from PIL import Image, ImageTk
import os
import threading

try:
    import winsound  # For playing sound on Windows
except ImportError:
    winsound = None

class ClickerObject:
    def __init__(self, name, base_cost, click_rate, game, image_path):
        self.name = name
        self.base_cost = base_cost
        self.click_rate = click_rate
        self.game = game
        self.quantity = 0
        self.level = 1
        self.image_path = image_path
        self.update_cost()
        self.update_upgrade_cost()
        self.running = False

    def buy(self):
        try:
            if self.game.points >= self.cost:
                self.game.points -= self.cost
                self.quantity += 1
                self.update_cost()
                self.update_upgrade_cost()
                self.game.update_passive_production_label()
                self.game.update_quantity_labels()
                self.game.add_clicker_image(self.name)
                self.game.play_sound('purchase')
                if not self.running:
                    self.running = True
                    self.auto_click()
        except Exception as e:
            print(f"Error in buy function for {self.name}: {e}")

    def update_cost(self):
        self.cost = self.base_cost * (1.1 ** self.quantity)  # Cost increases by 10% for each additional quantity

    def update_upgrade_cost(self):
        self.upgrade_cost = self.base_cost * (1.1 ** self.quantity) * 2  # Initial upgrade cost
        self.upgrade_cost *= 1.5 ** (self.quantity - 1)  # Increase by 50% for each upgrade

    def auto_click(self):
        if self.running:
            self.game.points += self.click_rate * self.quantity * self.level
            self.game.update_points_label()
            self.game.update_next_cost_label()
            self.game.master.after(1000, self.auto_click)

    def upgrade(self):
        if self.level < 50:  # Limit upgrades to 50 levels
            if self.game.points >= self.upgrade_cost:
                self.game.points -= self.upgrade_cost
                self.level += 1
                self.click_rate *= 1.5
                self.update_cost()
                self.update_upgrade_cost()  # Update the upgrade cost
                self.upgrade_cost *= 1.1  # Increase the upgrade cost by 10% after each upgrade
                self.game.update_points_label()
                self.game.update_quantity_labels()
                self.game.play_sound('upgrade')
                print(f"{self.name} upgraded to level {self.level}")
        else:
            print(f"{self.name} has reached the maximum level of 50")

class ClickerGame:
    def __init__(self, master):
        self.master = master
        self.master.title("Gym Clicker")

        self.points = 0
        self.objects = []
        self.clicker_images = {}
        self.objects_dict = {}

        self.create_widgets()
        self.load_sounds()

    def create_widgets(self):
        self.points_label = tk.Label(self.master, text="Músculo: 0")
        self.points_label.pack()

        self.passive_production_label = tk.Label(self.master, text="Produção Passiva: 0 Músculos/s")
        self.passive_production_label.pack()

        # Load and resize images
        self.image1 = Image.open("image1.png").resize((500, 500), Image.Resampling.LANCZOS)
        self.image2 = Image.open("image2.png").resize((500, 500), Image.Resampling.LANCZOS)

        self.image1_tk = ImageTk.PhotoImage(self.image1)
        self.image2_tk = ImageTk.PhotoImage(self.image2)
        self.current_image = self.image1_tk

        self.click_button = tk.Button(self.master, image=self.current_image, command=self.click)
        self.click_button.pack(side="left", padx=10, pady=10, anchor="nw")

        self.purchase_canvas = tk.Canvas(self.master)
        self.purchase_canvas.pack(side="right", padx=10, pady=10, anchor="nw", fill="both", expand=True)

        self.purchase_scrollbar = tk.Scrollbar(self.master, orient="vertical", command=self.purchase_canvas.yview)
        self.purchase_scrollbar.pack(side="right", fill="y")

        self.purchase_frame = tk.Frame(self.purchase_canvas)

        self.purchase_frame.bind(
            "<Configure>",
            lambda e: self.purchase_canvas.configure(
                scrollregion=self.purchase_canvas.bbox("all")
            )
        )

        self.purchase_canvas.create_window((0, 0), window=self.purchase_frame, anchor="nw")
        self.purchase_canvas.configure(yscrollcommand=self.purchase_scrollbar.set)

        self.canvas_frame = tk.Frame(self.master)
        self.canvas_frame.pack(side="left", padx=10, pady=10, anchor="nw")

        self.clicker_display_canvas = tk.Canvas(self.canvas_frame, width=600, height=600, bg="white")
        self.clicker_display_canvas.pack(pady=5, fill="both", expand=True)

        # Adding clicker objects
        self.add_object("Halter", 50, 1, "halter.png")
        self.add_object("Banco", 100, 2, "banco.png")
        self.add_object("Esteira", 300, 4, "esteira.png")
        self.add_object("Supino", 400, 8, "supino.png")
        self.add_object("Extensora", 600, 2.2 * 1.9 * 2.2 * 2.2, "extensora.png")

        self.update_points_label()
        self.update_next_cost_label()
        self.update_passive_production_label()
        self.update_quantity_labels()

    def click(self):
        self.points += 1
        self.update_points_label()
        self.play_sound('click')
        if self.current_image == self.image1_tk:
            self.current_image = self.image2_tk
        else:
            self.current_image = self.image1_tk
        self.click_button.config(image=self.current_image)

        # Button animation
        self.click_button.config(bg="lightblue")
        self.master.after(100, lambda: self.click_button.config(bg="SystemButtonFace"))

    def update_points_label(self):
        points_int = max(1, int(self.points))
        self.points_label.config(text=f"Músculo: {points_int:,}")

    def update_next_cost_label(self):
        for obj in self.objects:
            formatted_next_cost = "{:,}".format(int(obj.cost))
            obj.next_cost_label.config(text=f"Próximo {obj.name}: {formatted_next_cost} Músculos")

    def update_passive_production_label(self):
        passive_production = sum(obj.click_rate * obj.quantity * obj.level for obj in self.objects)
        passive_production_rounded = round(passive_production)
        self.passive_production_label.config(text=f"Produção Passiva: {passive_production_rounded} Músculos/s")

    def update_quantity_labels(self):
        for obj in self.objects:
            obj.quantity_label.config(text=f"Quantidade de {obj.name}: {obj.quantity} (Nível {obj.level})")
            upgrade_cost_formatted = "{:,}".format(int(obj.upgrade_cost))
            obj.upgrade_cost_label.config(text=f"Upgrade {obj.name}: {upgrade_cost_formatted} Músculos")

    def add_object(self, name, base_cost, click_rate, image_path):
        clicker_object = ClickerObject(name, base_cost, click_rate, self, image_path)
        self.objects.append(clicker_object)

        frame = tk.Frame(self.purchase_frame)
        frame.pack(padx=10, pady=10, anchor="nw")

        object_image = Image.open(image_path).resize((150, 150), Image.Resampling.LANCZOS)
        object_image_tk = ImageTk.PhotoImage(object_image)

        button = tk.Button(frame, text=" ", image=object_image_tk, compound="left", command=clicker_object.buy)
        button.image = object_image_tk
        button.pack()

        clicker_object.quantity_label = tk.Label(frame, text=f"Quantidade de {name}: {clicker_object.quantity}")
        clicker_object.quantity_label.pack()

        formatted_next_cost = "{:,}".format(int(clicker_object.cost))
        clicker_object.next_cost_label = tk.Label(frame, text=f"Próximo {name}: {formatted_next_cost} Músculos")
        clicker_object.next_cost_label.pack()

        upgrade_cost_formatted = "{:,}".format(int(clicker_object.upgrade_cost))
        clicker_object.upgrade_cost_label = tk.Label(frame, text=f"Upgrade {name}: {upgrade_cost_formatted} Músculos")
        clicker_object.upgrade_cost_label.pack()

        upgrade_button = tk.Button(frame, text=f"Upgrade {name}", command=clicker_object.upgrade)
        upgrade_button.pack()

        self.objects_dict[name] = clicker_object

    def add_clicker_image(self, name):
        try:
            image = Image.open(self.objects_dict[name].image_path).resize((150, 150), Image.Resampling.LANCZOS)
            image_tk = ImageTk.PhotoImage(image)

            canvas = self.clicker_display_canvas
            if name in self.clicker_images:
                rect_id, images = self.clicker_images[name]
                if len(images) < 15:
                    existing_images = len(images)
                    row = existing_images // 3
                    col = existing_images % 3
                    x_offset = col * 150
                    y_offset = row * 150
                    new_image = canvas.create_image(x_offset, y_offset, anchor="nw", image=image_tk)
                    images.append(new_image)
                    self.clicker_images[name] = (rect_id, images)
            else:
                rect_id = canvas.create_rectangle(0, 0, 450, 450, outline="black")
                image_item = canvas.create_image(0, 0, anchor="nw", image=image_tk)
                self.clicker_images[name] = (rect_id, [image_item])

            # Store the image reference to avoid garbage collection
            canvas.image_tk = image_tk
        except Exception as e:
            print(f"Error in add_clicker_image for {name}: {e}")

    def load_sounds(self):
        self.sounds = {
            'click': None,
            'purchase': None,
            'upgrade': None
        }
        if winsound:
            sounds_dir = "sounds"
            try:
                self.sounds['click'] = os.path.join(sounds_dir, "click.wav")
                self.sounds['purchase'] = os.path.join(sounds_dir, "purchase.wav")
                self.sounds['upgrade'] = os.path.join(sounds_dir, "upgrade.wav")
            except Exception as e:
                print(f"Error loading sounds: {e}")

    def play_sound(self, sound_name):
        if winsound and self.sounds[sound_name]:
            winsound.PlaySound(self.sounds[sound_name], winsound.SND_ASYNC)

def run_game():
    root = tk.Tk()
    game = ClickerGame(root)
    root.mainloop()

if __name__ == "__main__":
    run_game()
