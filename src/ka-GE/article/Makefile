# Variables
PANDOC = pandoc
SRC_DIR = .
BUILD_DIR = build
FLAGS = --pdf-engine=xelatex -V geometry:margin=2.5cm -V mainfont="DejaVu Serif" --filter pandoc-crossref

# Find all markdown files in the source directory
SOURCES = $(wildcard $(SRC_DIR)/*.md)
# Define the output PDF paths inside the build directory
OBJECTS = $(patsubst $(SRC_DIR)/%.md, $(BUILD_DIR)/%.pdf, $(SOURCES))

# Default target
all: $(BUILD_DIR) $(OBJECTS)

# Create the build directory if it doesn't exist
$(BUILD_DIR):
	mkdir -p $(BUILD_DIR)

# Rule to compile a PDF from a Markdown file
$(BUILD_DIR)/%.pdf: $(SRC_DIR)/%.md
	$(PANDOC) $< -o $@ $(FLAGS)

# Clean up generated files
clean:
	rm -rf $(BUILD_DIR)

.PHONY: all clean
