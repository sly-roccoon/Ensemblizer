cmake_minimum_required(VERSION 3.25) # at least

project(custom_name VERSION 0.0.0)

# PLUGIN DATA
set(PLUGIN_NAME "custom_name" CACHE STRING "Plugin name")
set(MANUFACT_CODE "ABCD" CACHE STRING "Plugin manufacturer code (4 characters, first capitalised)")
set(PLUGIN_CODE "Efgh" CACHE STRING "Plugin code (4 characters, first capitalised)")
##############

option(AUTOCOPY_PLUGIN "Copy VSTs/AUs to the system plugin directory after building" TRUE)

if(WIN32) # Windows Copy after install workaround
	set(VST3_INSTALL_DIR "$ENV{LocalAppData}/Programs/Common/VST3" CACHE PATH "VST3 installation directory")
	set(JUCE_COPY_DIR_ARGS VST3_COPY_DIR "${VST3_INSTALL_DIR}")
endif()

# JUCE lives in external/juce as a git submodule.
if(NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/external/juce/CMakeLists.txt")
	message(FATAL_ERROR
		"JUCE submodule is missing or uninitialised.\n"
		"Run: git submodule update --init --recursive")
endif()

add_subdirectory(external/juce) # NOT include() - that is for .cmake script files

juce_add_plugin(${PROJECT_NAME}
    # VERSION ...                               # Set this if the plugin version is different to the project version
    # ICON_BIG ...                              # ICON_* arguments specify a path to an image file to use as an icon for the Standalone
    # ICON_SMALL ...
    # COMPANY_NAME ...                          # Specify the name of the plugin's author
    # IS_SYNTH TRUE/FALSE                       # Is this a synth or an effect?
    # NEEDS_MIDI_INPUT TRUE/FALSE               # Does the plugin need midi input?
    # NEEDS_MIDI_OUTPUT TRUE/FALSE              # Does the plugin need midi output?
    # IS_MIDI_EFFECT TRUE/FALSE                 # Is this plugin a MIDI effect?
    # EDITOR_WANTS_KEYBOARD_FOCUS TRUE/FALSE    # Does the editor need keyboard focus?
    ${JUCE_COPY_DIR_ARGS}                       # Windows only: VST3_COPY_DIR override; expands to nothing elsewhere
    COPY_PLUGIN_AFTER_BUILD "${AUTOCOPY_PLUGIN}"  # Should the plugin be installed to a default location after building?
    PLUGIN_MANUFACTURER_CODE "${MANUFACT_CODE}"   # A four-character manufacturer id with at least one upper-case character //TODO: Change!
    PLUGIN_CODE "${PLUGIN_CODE}"					# A unique four-character plugin id with exactly one upper-case character //TODO: Change!
                                                # GarageBand 10.3 requires the first letter to be upper-case, and the remaining letters to be lower-case
    FORMATS VST3 Standalone                 	# The formats to build. Other valid formats are: AAX Unity VST AU AUv3
    PRODUCT_NAME "${PLUGIN_NAME}"		        # The name of the final executable, which can differ from the target name
    NEEDS_WEB_BROWSER FALSE                     # Set to TRUE if you need to use webviews
    NEEDS_CURL FALSE                            # Set to TRUE if you need to use curl
)
    
file(GLOB_RECURSE PROJECT_SOURCES CONFIGURE_DEPENDS "src/*.cpp")
target_sources(${PROJECT_NAME} PRIVATE ${PROJECT_SOURCES})

# On the target rather than via CMAKE_CXX_STANDARD, so it also applies when
# configuring without a preset (plain `cmake -S . -B build`) and propagates properly.
target_compile_features(${PROJECT_NAME} PUBLIC cxx_std_23)

target_compile_definitions(${PROJECT_NAME} # more juce settings
	PUBLIC
		JUCE_WEB_BROWSER=0
		JUCE_USE_CURL=0
		JUCE_VST3_CAN_REPLACE_VST2=0
)

target_link_libraries(${PROJECT_NAME}
	PRIVATE
		juce::juce_audio_utils
		juce::juce_dsp
	PUBLIC
		juce::juce_recommended_config_flags
		juce::juce_recommended_lto_flags
		juce::juce_recommended_warning_flags
)