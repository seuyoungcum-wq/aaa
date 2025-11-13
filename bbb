#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
Minesweeper (지뢰찾기) - tkinter 기반
- Left click: open cell
- Right click: toggle flag
- First click is always safe (mines placed after first click)
- Chording: click a revealed number when flagged neighbors == number to open neighbors
"""

import tkinter as tk
from tkinter import ttk, messagebox
import random
import time

# Configuration defaults
DEFAULT_ROWS = 9
DEFAULT_COLS = 9
DEFAULT_MINES = 10
CELL_SIZE = 30  # pixel (not strictly needed for Button-based UI)

class Minesweeper(tk.Tk):
    def __init__(self):
        super().__init__()
        self.title("지뢰찾기 (Minesweeper)")
        self.resizable(False, False)
        self._create_controls()
        self.game_frame = ttk.Frame(self)
        self.game_frame.grid(row=1, column=0, padx=8, pady=8)
        self._init_game(DEFAULT_ROWS, DEFAULT_COLS, DEFAULT_MINES)

    def _create_controls(self):
        ctrl = ttk.Frame(self, padding=(8,8))
        ctrl.grid(row=0, column=0, sticky="w")

        ttk.Label(ctrl, text="Rows:").grid(row=0, column=0, padx=(0,4))
        self.rows_var = tk.IntVar(value=DEFAULT_ROWS)
        self.rows_spin = ttk.Spinbox(ctrl, from_=5, to=30, width=5, textvariable=self.rows_var)
        self.rows_spin.grid(row=0, column=1)

        ttk.Label(ctrl, text="Cols:").grid(row=0, column=2, padx=(8,4))
        self.cols_var = tk.IntVar(value=DEFAULT_COLS)
        self.cols_spin = ttk.Spinbox(ctrl, from_=5, to=30, width=5, textvariable=self.cols_var)
        self.cols_spin.grid(row=0, column=3)

        ttk.Label(ctrl, text="Mines:").grid(row=0, column=4, padx=(8,4))
        self.mines_var = tk.IntVar(value=DEFAULT_MINES)
        self.mines_spin = ttk.Spinbox(ctrl, from_=1, to=200, width=7, textvariable=self.mines_var)
        self.mines_spin.grid(row=0, column=5)

        self.new_btn = ttk.Button(ctrl, text="New Game", command=self.new_game)
        self.new_btn.grid(row=0, column=6, padx=(12,0))

        # Status: mines left and timer
        status = ttk.Frame(ctrl)
        status.grid(row=0, column=7, padx=(12,0))
        self.mines_left_var = tk.IntVar(value=DEFAULT_MINES)
        self.timer_var = tk.StringVar(value="00:00")
        ttk.Label(status, text="Mines:").grid(row=0, column=0)
        self.mines_left_lbl = ttk.Label(status, textvariable=self.mines_left_var, width=4, anchor="center")
        self.mines_left_lbl.grid(row=0, column=1, padx=(2,10))
        ttk.Label(status, text="Time:").grid(row=0, column=2)
        self.timer_lbl = ttk.Label(status, textvariable=self.timer_var, width=6, anchor="center")
        self.timer_lbl.grid(row=0, column=3, padx=(2,0))

    def _init_game(self, rows, cols, mines):
        # model
        self.rows = rows
        self.cols = cols
        self.total_mines = min(max(1, mines), rows * cols - 1)
        self.mines_left_var.set(self.total_mines)

        self.mines = [[False]*cols for _ in range(rows)]
        self.numbers = [[0]*cols for _ in range(rows)]
        self.revealed = [[False]*cols for _ in range(rows)]
        self.flagged = [[False]*cols for _ in range(rows)]
        self.buttons = [[None]*cols for _ in range(rows)]

        self.started = False
        self.game_over = False
        self.start_time = None
        self.elapsed = 0
        self._stop_timer = False

        # Build UI grid
        for widget in self.game_frame.winfo_children():
            widget.destroy()

        for r in range(rows):
            for c in range(cols):
                b = tk.Button(self.game_frame, width=3, height=1, relief="raised", bg="lightgray",
                              command=lambda rr=r, cc=c: self.on_left(rr, cc))
                b.grid(row=r, column=c, padx=1, pady=1)
                b.bind("<Button-3>", lambda e, rr=r, cc=c: self.on_right(rr, cc))
                b.bind("<Double-Button-1>", lambda e, rr=r, cc=c: self.on_double(rr, cc))
                self.buttons[r][c] = b

        # Adjust window size
        self.update_idletasks()
        # Reset timer label
        self.timer_var.set("00:00")

    def new_game(self):
        try:
            rows = int(self.rows_var.get())
            cols = int(self.cols_var.get())
            mines = int(self.mines_var.get())
        except Exception:
            messagebox.showwarning("Invalid", "행/열/지뢰 수를 올바르게 입력하세요.")
            return
        if mines >= rows * cols:
            messagebox.showwarning("Invalid", "지뢰 수는 칸 수보다 작아야 합니다.")
            return
        self._stop_timer = True
        self._init_game(rows, cols, mines)

    def place_mines(self, safe_r, safe_c):
        # Place mines randomly avoiding (safe_r, safe_c) and its neighbors
        coords = [(r,c) for r in range(self.rows) for c in range(self.cols)]
        forbidden = set()
        for dr in (-1,0,1):
            for dc in (-1,0,1):
                rr = safe_r + dr
                cc = safe_c + dc
                if 0 <= rr < self.rows and 0 <= cc < self.cols:
                    forbidden.add((rr,cc))
        possible = [p for p in coords if p not in forbidden]
        mines_to_place = self.total_mines
        chosen = random.sample(possible, mines_to_place)
        for r,c in chosen:
            self.mines[r][c] = True

        # compute numbers
        for r in range(self.rows):
            for c in range(self.cols):
                if self.mines[r][c]:
                    self.numbers[r][c] = -1
                else:
                    cnt = 0
                    for dr in (-1,0,1):
                        for dc in (-1,0,1):
                            rr = r+dr; cc = c+dc
                            if 0 <= rr < self.rows and 0 <= cc < self.cols:
                                if self.mines[rr][cc]:
                                    cnt += 1
                    self.numbers[r][c] = cnt

    def on_left(self, r, c):
        if self.game_over:
            return
        if not self.started:
            # start game and place mines guaranteeing safe first click
            self.place_mines(r, c)
            self.started = True
            self.start_time = time.time()
            self._stop_timer = False
            self.update_timer()
        if self.flagged[r][c] or self.revealed[r][c]:
            # If revealed and number, try chord
            if self.revealed[r][c] and self.numbers[r][c] > 0:
                self.try_chord(r, c)
            return
        if self.mines[r][c]:
            # hit mine -> lose
            self.reveal_mine_hit(r, c)
            self.end_game(False)
            return
        self.reveal_cell(r, c)
        if self.check_win():
            self.end_game(True)

    def on_right(self, r, c):
        if self.game_over or not self.started and self.revealed[r][c]:
            return "break"
        if self.revealed[r][c]:
            return "break"
        # toggle flag
        self.flagged[r][c] = not self.flagged[r][c]
        b = self.buttons[r][c]
        if self.flagged[r][c]:
            b.config(text="⚑", fg="red")
        else:
            b.config(text="", fg="black")
        # update mines left display
        flags_count = sum(self.flagged[r2][c2] for r2 in range(self.rows) for c2 in range(self.cols))
        self.mines_left_var.set(max(0, self.total_mines - flags_count))
        return "break"  # prevent default context menu

    def on_double(self, r, c):
        # treat as chord attempt (like clicking revealed number)
        if self.revealed[r][c] and self.numbers[r][c] > 0:
            self.try_chord(r, c)

    def try_chord(self, r, c):
        num = self.numbers[r][c]
        flagged_neighbors = 0
        neighbors = []
        for dr in (-1,0,1):
            for dc in (-1,0,1):
                rr = r+dr; cc = c+dc
                if 0 <= rr < self.rows and 0 <= cc < self.cols:
                    if rr==r and cc==c:
                        continue
                    neighbors.append((rr,cc))
                    if self.flagged[rr][cc]:
                        flagged_neighbors += 1
        if flagged_neighbors == num:
            for rr,cc in neighbors:
                if not self.flagged[rr][cc] and not self.revealed[rr][cc]:
                    if self.mines[rr][cc]:
                        self.reveal_mine_hit(rr, cc)
                        self.end_game(False)
                        return
                    self.reveal_cell(rr, cc)
            if self.check_win():
                self.end_game(True)

    def reveal_cell(self, r, c):
        if self.revealed[r][c] or self.flagged[r][c]:
            return
        self.revealed[r][c] = True
        b = self.buttons[r][c]
        b.config(relief="sunken", bg="lightgrey")
        val = self.numbers[r][c]
        if val > 0:
            color = {
                1: "blue", 2: "green", 3: "red", 4: "darkblue",
                5: "darkred", 6: "cyan", 7: "black", 8: "gray"
            }.get(val, "black")
            b.config(text=str(val), fg=color, font=("TkDefaultFont", 10, "bold"))
        else:
            b.config(text="", fg="black")
            # flood fill neighbors
            for dr in (-1,0,1):
                for dc in (-1,0,1):
                    rr = r+dr; cc = c+dc
                    if 0 <= rr < self.rows and 0 <= cc < self.cols:
                        if not self.revealed[rr][cc] and not self.mines[rr][cc]:
                            self.reveal_cell(rr, cc)

    def reveal_mine_hit(self, r, c):
        # reveal this mine specially
        b = self.buttons[r][c]
        b.config(text="✹", bg="red", fg="white", relief="sunken")
        # reveal all mines
        for rr in range(self.rows):
            for cc in range(self.cols):
                if self.mines[rr][cc] and not self.flagged[rr][cc]:
                    btn = self.buttons[rr][cc]
                    btn.config(text="✹", bg="orange", fg="black")
                if self.flagged[rr][cc] and not self.mines[rr][cc]:
                    # wrongly flagged
                    btn = self.buttons[rr][cc]
                    btn.config(text="✖", bg="pink", fg="black")

    def end_game(self, won):
        self.game_over = True
        self._stop_timer = True
        self.reveal_all_flags()
        if won:
            elapsed = int(time.time() - self.start_time) if self.start_time else 0
            messagebox.showinfo("You Win!", f"축하합니다! 모든 지뢰를 찾았습니다.\n시간: {elapsed}초")
        else:
            messagebox.showinfo("Game Over", "지뢰를 밟았습니다. 다시 시도하세요.")

    def reveal_all_flags(self):
        # reveal flagged correctness and disable all buttons
        for r in range(self.rows):
            for c in range(self.cols):
                b = self.buttons[r][c]
                b.config(state="disabled")
                if self.flagged[r][c]:
                    if self.mines[r][c]:
                        b.config(text="⚑", fg="green")
                    else:
                        b.config(text="⚑", fg="red")
                else:
                    if self.mines[r][c] and not self.revealed[r][c]:
                        b.config(text="✹", fg="black")

    def check_win(self):
        # win when all non-mine cells are revealed
        for r in range(self.rows):
            for c in range(self.cols):
                if not self.mines[r][c] and not self.revealed[r][c]:
                    return False
        return True

    def update_timer(self):
        if self._stop_timer:
            return
        if not self.started:
            return
        self.elapsed = int(time.time() - self.start_time)
        mins = self.elapsed // 60
        secs = self.elapsed % 60
        self.timer_var.set(f"{mins:02d}:{secs:02d}")
        # schedule next update
        self.after(500, self.update_timer)


if __name__ == "__main__":
    app = Minesweeper()
    app.mainloop()
