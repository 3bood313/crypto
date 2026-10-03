import tkinter as tk
from tkinter import ttk, messagebox
import string

# Caesar
def caesar_encrypt(text, shift):
    result = []
    for ch in text.upper():
        if ch.isalpha():
            result.append(chr((ord(ch) - 65 + shift) % 26 + 65))
        else:
            result.append(ch)
    return ''.join(result)

def caesar_decrypt(text, shift):
    return caesar_encrypt(text, -shift)

# Vigenere
def vigenere_encrypt(text, key):
    key = key.upper()
    result, ki = [], 0
    for ch in text.upper():
        if ch.isalpha():
            shift = ord(key[ki % len(key)]) - 65
            result.append(chr((ord(ch) - 65 + shift) % 26 + 65))
            ki += 1
        else:
            result.append(ch)
    return ''.join(result)

def vigenere_decrypt(text, key):
    key = key.upper()
    result, ki = [], 0
    for ch in text.upper():
        if ch.isalpha():
            shift = ord(key[ki % len(key)]) - 65
            result.append(chr((ord(ch) - 65 - shift) % 26 + 65))
            ki += 1
        else:
            result.append(ch)
    return ''.join(result)

# Playfair
def playfair_matrix(key):
    key = key.upper().replace('J', 'I')
    seen, matrix = set(), []
    for ch in key + string.ascii_uppercase:
        if ch == 'J':
            continue
        if ch not in seen:
            seen.add(ch)
            matrix.append(ch)
    return [matrix[i*5:(i+1)*5] for i in range(5)]

def find_pos(matrix, ch):
    for r, row in enumerate(matrix):
        if ch in row:
            return r, row.index(ch)

def playfair_digraphs(text):
    text = text.upper().replace('J', 'I')
    text = ''.join(c for c in text if c.isalpha())
    pairs = []
    i = 0
    while i < len(text):
        a = text[i]
        if i + 1 == len(text):
            pairs.append((a, 'X'))
            i += 1
        elif text[i] == text[i+1]:
            pairs.append((a, 'X'))
            i += 1
        else:
            pairs.append((a, text[i+1]))
            i += 2
    return pairs

def playfair_encrypt(text, key):
    matrix = playfair_matrix(key)
    result = []
    for a, b in playfair_digraphs(text):
        r1, c1 = find_pos(matrix, a)
        r2, c2 = find_pos(matrix, b)
        if r1 == r2:
            result += [matrix[r1][(c1+1)%5], matrix[r2][(c2+1)%5]]
        elif c1 == c2:
            result += [matrix[(r1+1)%5][c1], matrix[(r2+1)%5][c2]]
        else:
            result += [matrix[r1][c2], matrix[r2][c1]]
    return ''.join(result)

def playfair_decrypt(text, key):
    matrix = playfair_matrix(key)
    result = []
    for a, b in playfair_digraphs(text):
        r1, c1 = find_pos(matrix, a)
        r2, c2 = find_pos(matrix, b)
        if r1 == r2:
            result += [matrix[r1][(c1-1)%5], matrix[r2][(c2-1)%5]]
        elif c1 == c2:
            result += [matrix[(r1-1)%5][c1], matrix[(r2-1)%5][c2]]
        else:
            result += [matrix[r1][c2], matrix[r2][c1]]

    # Remove padding X
    cleaned = []
    i = 0
    while i < len(result):
        cleaned.append(result[i])
        if (i + 2 < len(result)
                and result[i + 1] == 'X'
                and result[i] == result[i + 2]):
            i += 2
        else:
            i += 1
    if cleaned and cleaned[-1] == 'X':
        cleaned.pop()

    return ''.join(cleaned)


# ─────────────────────────────────────────────
#  GUI
# ─────────────────────────────────────────────
class CryptoApp(tk.Tk):
    CIPHERS = ["Playfair", "Caesar", "Vigenère"]

    BG        = "#0d0f14"
    CARD_BG   = "#131720"
    ACCENT    = "#00e5a0"
    ACCENT2   = "#0088ff"
    TEXT      = "#e8eaf0"
    MUTED     = "#5a6070"
    BORDER    = "#1e2535"
    TAB_ACT   = "#1a2035"

    FONT_TITLE  = ("Courier New", 22, "bold")
    FONT_LABEL  = ("Courier New", 10, "bold")
    FONT_BODY   = ("Courier New", 11)
    FONT_OUTPUT = ("Courier New", 12, "bold")
    FONT_TAB    = ("Courier New", 11, "bold")

    def __init__(self):
        super().__init__()
        self.title("CipherX")
        self.configure(bg=self.BG)
        self.geometry("740x680")
        self.resizable(False, False)
        self._current = tk.StringVar(value=self.CIPHERS[0])
        self._build_ui()

    # ── layout
    def _build_ui(self):
        # title bar
        header = tk.Frame(self, bg=self.BG, pady=18)
        header.pack(fill="x", padx=30)

        tk.Label(header, text="⬡ CIPHERX",
                font=self.FONT_TITLE, bg=self.BG,
                fg=self.ACCENT).pack(side="left")
        tk.Label(header, text="encrypt · decrypt",
                font=("Courier New", 10), bg=self.BG,
                fg=self.MUTED).pack(side="left", padx=14, pady=6)

        # separator
        tk.Frame(self, bg=self.BORDER, height=1).pack(fill="x", padx=30)

        # cipher selector tabs
        tab_row = tk.Frame(self, bg=self.BG, pady=16)
        tab_row.pack(fill="x", padx=30)
        tk.Label(tab_row, text="ALGORITHM:", font=self.FONT_LABEL,
                bg=self.BG, fg=self.MUTED).pack(side="left", padx=(0,12))
        self._tab_btns = {}
        for name in self.CIPHERS:
            btn = tk.Button(
                tab_row, text=name,
                font=self.FONT_TAB,
                relief="flat", cursor="hand2",
                bd=0, padx=18, pady=8,
                command=lambda n=name: self._select(n)
            )
            btn.pack(side="left", padx=4)
            self._tab_btns[name] = btn
        self._refresh_tabs()

        # main card
        self._card = tk.Frame(self, bg=self.CARD_BG,
                            highlightbackground=self.BORDER,
                            highlightthickness=1)
        self._card.pack(fill="both", expand=True, padx=30, pady=(0,20))
        self._draw_card()

    def _select(self, name):
        self._current.set(name)
        self._refresh_tabs()
        self._draw_card()

    def _refresh_tabs(self):
        for name, btn in self._tab_btns.items():
            active = name == self._current.get()
            btn.configure(
                bg=self.TAB_ACT if active else self.BG,
                fg=self.ACCENT  if active else self.MUTED,
                highlightbackground=self.ACCENT if active else self.BORDER,
                highlightthickness=1 if active else 0,
                relief="flat"
            )

    # ── card
    def _draw_card(self):
        for w in self._card.winfo_children():
            w.destroy()

        cipher = self._current.get()
        pad = dict(padx=28, pady=10)

        # cipher badge
        badge_row = tk.Frame(self._card, bg=self.CARD_BG)
        badge_row.pack(fill="x", **pad)
        badge_row.pack_configure(pady=(18,6))

        tk.Label(badge_row, text=f"[ {cipher.upper()} ]",
                font=("Courier New", 13, "bold"),
                bg=self.CARD_BG, fg=self.ACCENT2).pack(side="left")

        desc = {"Playfair":  "5×5 matrix · digraph substitution",
                "Caesar":    "shift cipher · single key integer",
                "Vigenère":  "polyalphabetic · keyword stream"}
        tk.Label(badge_row, text=desc[cipher],
                font=("Courier New", 9), bg=self.CARD_BG,
                fg=self.MUTED).pack(side="left", padx=12)

        tk.Frame(self._card, bg=self.BORDER, height=1).pack(
            fill="x", padx=28, pady=(0,6))

        # plaintext
        self._label(self._card, "PLAINTEXT / CIPHERTEXT INPUT")
        self._txt_in = self._text_box(self._card, height=4)

        # key field
        if cipher == "Caesar":
            self._label(self._card, "SHIFT  (integer, e.g. 3)")
            self._key_var = tk.StringVar(value="3")
            self._ent_key = self._entry(self._card, self._key_var)
        else:
            lbl = "KEYWORD" if cipher == "Vigenère" else "KEY PHRASE"
            self._label(self._card, lbl + "  (letters only)")
            self._key_var = tk.StringVar(value="KEY")
            self._ent_key = self._entry(self._card, self._key_var)

        # buttons
        btn_row = tk.Frame(self._card, bg=self.CARD_BG)
        btn_row.pack(fill="x", padx=28, pady=12)
        self._make_btn(btn_row, "▶  ENCRYPT", self.ACCENT,  self._do_encrypt).pack(side="left", padx=(0,10))
        self._make_btn(btn_row, "◀  DECRYPT", self.ACCENT2, self._do_decrypt).pack(side="left")

        # output
        tk.Frame(self._card, bg=self.BORDER, height=1).pack(
            fill="x", padx=28, pady=(4,0))
        self._label(self._card, "OUTPUT")
        self._txt_out = self._text_box(self._card, height=4, readonly=True)

        # copy button
        copy_row = tk.Frame(self._card, bg=self.CARD_BG)
        copy_row.pack(fill="x", padx=28, pady=(0,18))
        tk.Button(copy_row, text="⧉  copy to clipboard",
                font=("Courier New", 9), bg=self.CARD_BG,
                fg=self.MUTED, relief="flat", cursor="hand2",
                activebackground=self.CARD_BG, activeforeground=self.ACCENT,
                command=self._copy).pack(side="left")

    # ── helpers
    def _label(self, parent, text):
        tk.Label(parent, text=text, font=self.FONT_LABEL,
                bg=self.CARD_BG, fg=self.MUTED,
                anchor="w").pack(fill="x", padx=28, pady=(6,2))

    def _text_box(self, parent, height=4, readonly=False):
        frame = tk.Frame(parent, bg=self.BORDER, padx=1, pady=1)
        frame.pack(fill="x", padx=28, pady=(0,4))
        txt = tk.Text(frame, height=height, font=self.FONT_BODY,
                    bg="#0a0c12", fg=self.TEXT,
                    insertbackground=self.ACCENT,
                    relief="flat", bd=0, padx=10, pady=8,
                    selectbackground=self.ACCENT,
                    selectforeground="#000")
        if readonly:
            txt.configure(state="disabled", fg=self.ACCENT,
                        font=self.FONT_OUTPUT)
        txt.pack(fill="x")
        return txt

    def _entry(self, parent, var):
        frame = tk.Frame(parent, bg=self.BORDER, padx=1, pady=1)
        frame.pack(fill="x", padx=28, pady=(0,4))
        e = tk.Entry(frame, textvariable=var, font=self.FONT_BODY,
                    bg="#0a0c12", fg=self.ACCENT,
                    insertbackground=self.ACCENT,
                    relief="flat", bd=0)
        e.pack(fill="x", ipady=7, padx=10)
        return e

    def _make_btn(self, parent, text, color, cmd):
        return tk.Button(parent, text=text,
                        font=("Courier New", 11, "bold"),
                        bg=color, fg="#000", relief="flat",
                        cursor="hand2", padx=20, pady=8,
                        activebackground=self.ACCENT,
                        activeforeground="#000",
                        command=cmd)

    # ── actions ──────────────────────────────
    def _get_inputs(self):
        txt = self._txt_in.get("1.0", "end").strip()
        key = self._key_var.get().strip()
        if not txt:
            messagebox.showwarning("Empty input", "Please enter some text.")
            return None, None
        if not key:
            messagebox.showwarning("Empty key", "Please enter a key/shift.")
            return None, None
        return txt, key

    def _set_output(self, result):
        self._txt_out.configure(state="normal")
        self._txt_out.delete("1.0", "end")
        self._txt_out.insert("1.0", result)
        self._txt_out.configure(state="disabled")

    def _do_encrypt(self):
        txt, key = self._get_inputs()
        if txt is None:
            return
        cipher = self._current.get()
        try:
            if cipher == "Caesar":
                result = caesar_encrypt(txt, int(key))
            elif cipher == "Vigenère":
                result = vigenere_encrypt(txt, key)
            else:
                result = playfair_encrypt(txt, key)
            self._set_output(result)
        except Exception as e:
            messagebox.showerror("Error", str(e))

    def _do_decrypt(self):
        txt, key = self._get_inputs()
        if txt is None:
            return
        cipher = self._current.get()
        try:
            if cipher == "Caesar":
                result = caesar_decrypt(txt, int(key))
            elif cipher == "Vigenère":
                result = vigenere_decrypt(txt, key)
            else:
                result = playfair_decrypt(txt, key)
            self._set_output(result)
        except Exception as e:
            messagebox.showerror("Error", str(e))

    def _copy(self):
        out = self._txt_out.get("1.0", "end").strip()
        if out:
            self.clipboard_clear()
            self.clipboard_append(out)


if __name__ == "__main__":
    app = CryptoApp()
    app.mainloop()
