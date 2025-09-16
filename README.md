# calculate_screw_flight_dimensions
mport numpy as np
import matplotlib.pyplot as plt

def calculate_screw_flight_dimensions(outer_diameter, inner_diameter, pitch, material_thickness):
  """
  Calculates the flat pattern dimensions for a screw conveyor flight
  using the formulas R1=(L1*h)/(L1-L2)-t and R2=R1+h.

  Args:
    outer_diameter: The outer diameter of the screw flight.
    inner_diameter: The inner diameter of the screw flight.
    pitch: The axial distance between corresponding points on consecutive turns.
    material_thickness: The thickness of the material used for the flight.

  Returns:
    A dictionary containing the outer radius, inner radius, and sweep angle
    (in degrees) of the flat pattern.
  """
  # Implement basic validation for inputs
  if outer_diameter <= 0 or inner_diameter <= 0 or pitch <= 0:
      raise ValueError("Outer Diameter, Inner Diameter, and Pitch must be positive.")
  if inner_diameter >= outer_diameter:
      raise ValueError("Inner Diameter must be less than Outer Diameter.")
  if material_thickness < 0:
      raise ValueError("Material thickness cannot be negative.")


  # Calculate L1
  L1 = np.sqrt((np.pi * outer_diameter)**2 + pitch**2)

  # Calculate L2
  L2 = np.sqrt((np.pi * inner_diameter)**2 + pitch**2)

  # Calculate h
  h = (outer_diameter - inner_diameter) / 2

  # Calculate outer radius (R1)
  # Add a check for L1 == L2 to prevent division by zero
  if np.isclose(L1, L2): # Use np.isclose for floating point comparison
      raise ValueError("L1 and L2 are equal, which would result in division by zero. Check input values (Outer Diameter, Inner Diameter, and Pitch).")
  outer_radius_developed = (L1 * h) / (L1 - L2) - material_thickness

  # Calculate inner radius (R2)
  inner_radius_developed = outer_radius_developed - h

  # Implement basic validation for calculated results
  if outer_radius_developed <= 0:
      raise ValueError("Calculated outer developed radius is zero or negative. Check input values.")
  if inner_radius_developed <= 0:
      raise ValueError("Calculated inner developed radius is zero or negative. Check input values.")
  if inner_radius_developed >= outer_radius_developed:
      raise ValueError("Calculated inner developed radius is greater than or equal to outer developed radius. Check input values.")


  # Calculate the sweep angle in radians
  # The sweep angle is the angle subtended by the arc length L1 at the radius R1
  if outer_radius_developed <= 0: # Prevent division by zero if R1 is invalid
       raise ValueError("Cannot calculate sweep angle: Outer radius is zero or negative.")
  sweep_angle_radians = L1 / outer_radius_developed

  # Convert sweep angle to degrees
  sweep_angle_degrees = np.degrees(sweep_angle_radians)

  return {
      "outer_radius": outer_radius_developed,
      "inner_radius": inner_radius_developed,
      "sweep_angle_degrees": sweep_angle_degrees
  }

def generate_flight_pattern_data(outer_radius, inner_radius, sweep_angle_degrees):
    """
    Generates geometric data points for the developed screw conveyor flight pattern.

    Args:
        outer_radius: The outer radius of the developed flat pattern.
        inner_radius: The inner radius of the developed flat pattern.
        sweep_angle_degrees: The sweep angle of the developed flat pattern in degrees.

    Returns:
        A dictionary containing the x and y coordinates for the outer arc,
        inner arc, and connecting lines.
    """
    # Convert sweep angle to radians
    sweep_angle_radians = np.radians(sweep_angle_degrees)

    # Generate angles for the arcs
    angles = np.linspace(0, sweep_angle_radians, 100) # Use 100 points for smooth arcs

    # Calculate coordinates for the outer arc
    outer_arc_x = outer_radius * np.cos(angles)
    outer_arc_y = outer_radius * np.sin(angles)

    # Calculate coordinates for the inner arc
    inner_arc_x = inner_radius * np.cos(angles)
    inner_arc_y = inner_radius * np.sin(angles)

    # Create coordinates for the connecting lines
    # Line 1: Connects start points (at angle 0)
    line1_x = [outer_radius * np.cos(0), inner_radius * np.cos(0)]
    line1_y = [outer_radius * np.sin(0), inner_radius * np.sin(0)]

    # Line 2: Connects end points (at sweep_angle_radians)
    line2_x = [outer_radius * np.cos(sweep_angle_radians), inner_radius * np.cos(sweep_angle_radians)]
    line2_y = [outer_radius * np.sin(sweep_angle_radians), inner_radius * np.sin(sweep_angle_radians)]

    return {
        "outer_arc_x": outer_arc_x,
        "outer_arc_y": outer_arc_y,
        "inner_arc_x": inner_arc_x,
        "inner_arc_y": inner_arc_y,
        "line1_x": line1_x,
        "line1_y": line1_y,
        "line2_x": line2_x,
        "line2_y": line2_y,
    }


# Check if the calculate button is clicked
if calculate_button:
    try:
        # Perform the calculation using the input values
        flight_dimensions = calculate_screw_flight_dimensions(outer_diameter, inner_diameter, pitch, material_thickness)

        # Display the calculated dimensions
        results_placeholder.success("Calculation Successful!")
        results_placeholder.write("Calculated Dimensions:")
        results_placeholder.write(f"Outer Radius: {flight_dimensions['outer_radius']:.2f} mm")
        results_placeholder.write(f"Inner Radius: {flight_dimensions['inner_radius']:.2f} mm")
        results_placeholder.write(f"Sweep Angle: {flight_dimensions['sweep_angle_degrees']:.2f} degrees")

        # Generate geometric data for plotting
        flight_pattern_data = generate_flight_pattern_data(
            flight_dimensions['outer_radius'],
            flight_dimensions['inner_radius'],
            flight_dimensions['sweep_angle_degrees']
        )

        # Create a new figure and axes for the plot
        fig, ax = plt.subplots(figsize=(8, 8)) # Adjust figure size as needed

        # Plot the outer arc
        ax.plot(flight_pattern_data["outer_arc_x"], flight_pattern_data["outer_arc_y"], label="Outer Arc", color='blue')

        # Plot the inner arc
        ax.plot(flight_pattern_data["inner_arc_x"], flight_pattern_data["inner_arc_y"], label="Inner Arc", color='red')

        # Plot the first connecting line
        ax.plot(flight_pattern_data["line1_x"], flight_pattern_data["line1_y"], label="Connecting Line 1", color='green')

        # Plot the second connecting line
        ax.plot(flight_pattern_data["line2_x"], flight_pattern_data["line2_y"], label="Connecting Line 2", color='purple')

        # Ensure the aspect ratio of the plot is equal
        ax.set_aspect('equal', adjustable='box')

        # Add a title to the plot
        ax.set_title("Developed Screw Conveyor Flight Pattern")

        # Add labels to the x and y axes
        ax.set_xlabel("X-coordinate (mm)")
        ax.set_ylabel("Y-coordinate (mm)")

        # Add a grid to the plot
        ax.grid(True)

        # Add a legend
        ax.legend()

        # Display the plot in Streamlit
        plot_placeholder.pyplot(fig)

    except ValueError as ve:
        # Display input errors
        results_placeholder.error(f"Input Error: {ve}")
        plot_placeholder.empty() # Clear the plot area on error
    except Exception as e:
        # Display any other unexpected errors
        results_placeholder.error(f"An unexpected error occurred: {e}")
        plot_placeholder.empty() # Clear the plot area on error
