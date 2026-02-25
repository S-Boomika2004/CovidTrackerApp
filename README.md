import pandas as pd
import tkinter as tk
from tkinter import filedialog, ttk, messagebox
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg
import matplotlib.pyplot as plt

class CovidTrackerApp:
    def __init__(self, root):
        self.root = root
        self.root.configure(bg = 'black')
        self.root.title("COVID-19 Country-Wise Data Tracker")
        self.root.geometry("800x600")
        self.df = None

        self.create_widgets()

    def create_widgets(self):
        # === Top Frame ===
        frame_top = tk.Frame(self.root, pady=10)
        frame_top.pack(fill="x")

        btn_upload = tk.Button(frame_top, text="Upload CSV", command=self.load_data)
        btn_upload.pack(side="left", padx=10)

        self.cmb_country = ttk.Combobox(frame_top, state="readonly", width=30)
        self.cmb_country.pack(side="left", padx=10)

        btn_show = tk.Button(frame_top, text="Show Country Data", command=self.show_country_data)
        btn_show.pack(side="left", padx=10)

        # === Summary Frame ===
        frame_summary = tk.LabelFrame(self.root, text="Country Summary", padx=10, pady=10)
        frame_summary.pack(fill="x", padx=10, pady=5)

        self.lbl_confirmed = ttk.Label(frame_summary, text="Total Confirmed: ", font=("Arial", 12))
        self.lbl_confirmed.pack(anchor="w")
        self.lbl_deaths = ttk.Label(frame_summary, text="Total Deaths: ", font=("Arial", 12))
        self.lbl_deaths.pack(anchor="w")
        self.lbl_recovered = ttk.Label(frame_summary, text="Total Recovered: ", font=("Arial", 12))
        self.lbl_recovered.pack(anchor="w")
        self.lbl_active = ttk.Label(frame_summary, text="Active Cases: ", font=("Arial", 12))
        self.lbl_active.pack(anchor="w")

        # === Plot Frame ===
        frame_plot = tk.LabelFrame(self.root, text="Visualization", padx=10, pady=10)
        frame_plot.pack(fill="both", expand=True, padx=10, pady=5)

        self.figure, self.ax = plt.subplots(figsize=(6, 4))
        self.canvas = FigureCanvasTkAgg(self.figure, master=frame_plot)
        self.canvas.get_tk_widget().pack(fill="both", expand=True)

    def load_data(self):
        """Load CSV dataset"""
        file_path = filedialog.askopenfilename(
            title="Select COVID-19 Dataset",
            filetypes=[("CSV Files", "*.csv")]
        )
        if not file_path:
            return

        try:
            self.df = pd.read_csv(file_path)

            # Clean columns
            self.df.columns = [c.strip() for c in self.df.columns]

            if "Country/Region" not in self.df.columns:
                messagebox.showerror("Error", "CSV must have 'Country/Region' column!")
                return

            countries = sorted(self.df["Country/Region"].dropna().unique())
            self.cmb_country["values"] = countries
            self.cmb_country.current(0)
            messagebox.showinfo("Success", "Dataset loaded successfully!")

        except Exception as e:
            messagebox.showerror("Error", f"Failed to load file:\n{e}")

    def show_country_data(self):
        """Show selected country's data"""
        if self.df is None:
            messagebox.showwarning("Warning", "Please upload a dataset first.")
            return

        country = self.cmb_country.get()
        if not country:
            messagebox.showwarning("Warning", "Please select a country.")
            return

        row = self.df[self.df["Country/Region"] == country].iloc[0]

        confirmed = row.get("Confirmed", 0)
        deaths = row.get("Deaths", 0)
        recovered = row.get("Recovered", 0)
        active = row.get("Active", 0)

        # Update labels
        self.lbl_confirmed.config(text=f"Total Confirmed: {int(confirmed):,}")
        self.lbl_deaths.config(text=f"Total Deaths: {int(deaths):,}")
        self.lbl_recovered.config(text=f"Total Recovered: {int(recovered):,}")
        self.lbl_active.config(text=f"Active Cases: {int(active):,}")

        # Plot bar chart
        self.ax.clear()
        data = {"Confirmed": confirmed, "Deaths": deaths, "Recovered": recovered, "Active": active}
        self.ax.bar(data.keys(), data.values(), color=["blue", "red", "green", "orange"], alpha=0.6)
        self.ax.set_title(f"COVID-19 Summary: {country}")
        self.ax.set_ylabel("Cases Count")

        self.canvas.draw()


if __name__ == "__main__":
    root = tk.Tk()
    app = CovidTrackerApp(root)
    root.mainloop()

