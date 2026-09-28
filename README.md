# unit-conversion-program
The Unit Conversion Application is a software utility that allows users to instantly convert numerical values from one measurement unit to another across multiple measurement domains (e.g., converting kilometers to miles, or Celsius to Fahrenheit). The application handles complex, non-linear conversions (such as temperature) as well as linear 
import tkinter as tk
from tkinter import ttk, messagebox

class UnitConverterApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Advanced Unit Converter")
        self.root.geometry("450x400")
        self.root.resizable(False, False)
        
        # Define unit dictionaries with scaling factors (relative to a base unit)
        self.conversion_data = {
            "Length": {
                "Base": "Meters",
                "Units": {
                    "Meters": 1.0,
                    "Kilometers": 1000.0,
                    "Centimeters": 0.01,
                    "Millimeters": 0.001,
                    "Miles": 1609.34,
                    "Yards": 0.9144,
                    "Feet": 0.3048,
                    "Inches": 0.0254
                }
            },
            "Weight": {
                "Base": "Kilograms",
                "Units": {
                    "Kilograms": 1.0,
                    "Grams": 0.001,
                    "Milligrams": 0.000001,
                    "Pounds (lbs)": 0.45359237,
                    "Ounces (oz)": 0.02834952
                }
            },
            "Temperature": {
                "Base": "Special",
                "Units": ["Celsius", "Fahrenheit", "Kelvin"]
            }
        }
        
        self.create_widgets()
        
    def create_widgets(self):
        # Main container padding
        main_frame = ttk.Frame(self.root, padding="20")
        main_frame.pack(fill=tk.BOTH, expand=True)
        
        # 1. Category Selection
        ttk.Label(main_frame, text="Select Category:", font=("Arial", 11, "bold")).grid(row=0, column=0, sticky=tk.W, pady=5)
        self.category_var = tk.StringVar()
        self.category_combobox = ttk.Combobox(main_frame, textvariable=self.category_var, state="readonly", font=("Arial", 10))
        self.category_combobox['values'] = list(self.conversion_data.keys())
        self.category_combobox.grid(row=0, column=1, columnspan=2, sticky=tk.EW, pady=5)
        self.category_combobox.bind("<<ComboboxSelected>>", self.update_units)
        
        # Separator line
        ttk.Separator(main_frame, orient='horizontal').grid(row=1, column=0, columnspan=3, sticky=tk.EW, pady=15)
        
        # 2. Input Value Section
        ttk.Label(main_frame, text="Enter Value:", font=("Arial", 10)).grid(row=2, column=0, sticky=tk.W, pady=5)
        self.input_entry = ttk.Entry(main_frame, font=("Arial", 10))
        self.input_entry.grid(row=2, column=1, columnspan=2, sticky=tk.EW, pady=5)
        
        # 3. 'From' Unit Dropdown
        ttk.Label(main_frame, text="From Unit:", font=("Arial", 10)).grid(row=3, column=0, sticky=tk.W, pady=5)
        self.from_unit_var = tk.StringVar()
        self.from_unit_combobox = ttk.Combobox(main_frame, textvariable=self.from_unit_var, state="readonly", font=("Arial", 10))
        self.from_unit_combobox.grid(row=3, column=1, columnspan=2, sticky=tk.EW, pady=5)
        
        # 4. 'To' Unit Dropdown
        ttk.Label(main_frame, text="To Unit:", font=("Arial", 10)).grid(row=4, column=0, sticky=tk.W, pady=5)
        self.to_unit_var = tk.StringVar()
        self.to_unit_combobox = ttk.Combobox(main_frame, textvariable=self.to_unit_var, state="readonly", font=("Arial", 10))
        self.to_unit_combobox.grid(row=4, column=1, columnspan=2, sticky=tk.EW, pady=5)
        
        # 5. Action Button
        self.convert_btn = ttk.Button(main_frame, text="Convert", command=self.perform_conversion)
        self.convert_btn.grid(row=5, column=0, columnspan=3, sticky=tk.EW, pady=20)
        
        # 6. Result Display Area
        self.result_var = tk.StringVar(value="Result: ")
        self.result_label = ttk.Label(main_frame, textvariable=self.result_var, font=("Arial", 12, "bold"), foreground="#2e7d32")
        self.result_label.grid(row=6, column=0, columnspan=3, pady=10)
        
        # Initialize UI state with the first item
        self.category_combobox.current(0)
        self.update_units()

    def update_units(self, event=None):
        """Triggered whenever the user switches categories to reload matching units."""
        category = self.category_var.get()
        data = self.conversion_data[category]
        
        if category == "Temperature":
            unit_list = data["Units"]
        else:
            unit_list = list(data["Units"].keys())
            
        self.from_unit_combobox['values'] = unit_list
        self.to_unit_combobox['values'] = unit_list
        
        # Set default selections to prevent empty dropdown errors
        self.from_unit_combobox.current(0)
        self.to_unit_combobox.current(1 if len(unit_list) > 1 else 0)
        self.result_var.set("Result: ")

    def convert_temperature(self, value, from_u, to_u):
        """Dedicated logic for handling non-linear temperature metrics."""
        # Step 1: Base absolute normalization (Convert everything to Celsius first)
        if from_u == "Fahrenheit":
            celsius_val = (value - 32) * 5/9
        elif from_u == "Kelvin":
            celsius_val = value - 273.15
        else:
            celsius_val = value
            
        # Step 2: Convert from baseline Celsius out to the target metric
        if to_u == "Fahrenheit":
            return (celsius_val * 9/5) + 32
        elif to_u == "Kelvin":
            return celsius_val + 273.15
        return celsius_val

    def perform_conversion(self):
        """Processes arithmetic mapping or logic flags depending on chosen fields."""
        try:
            # Validate input entry string yields a valid floating-point number
            raw_input = self.input_entry.get().strip()
            if not raw_input:
                raise ValueError("Input line cannot be empty.")
            
            value = float(raw_input)
        except ValueError:
            messagebox.showerror("Invalid Input", "Please enter a valid numeric value to convert.")
            return

        category = self.category_var.get()
        from_u = self.from_unit_var.get()
        to_u = self.to_unit_var.get()
        
        # Handle structural conversion paths
        if category == "Temperature":
            result = self.convert_temperature(value, from_u, to_u)
        else:
            # Linear multiplier logic (Value * Source Factor / Destination Factor)
            unit_dict = self.conversion_data[category]["Units"]
            value_in_base = value * unit_dict[from_u]
            result = value_in_base / unit_dict[to_u]
            
        # Format output smoothly to prevent long trailing floats
        if result.is_integer():
            formatted_result = f"{int(result)}"
        else:
            formatted_result = f"{result:.4f}".rstrip('0').rstrip('.')
            
        self.result_var.set(f"Result: {value} {from_u} = {formatted_result} {to_u}")

if __name__ == "__main__":
    root = tk.Tk()
    app = UnitConverterApp(root)
    root.mainloop()
