#include "vst2.x/aeffectx.h"

// Phaser Plugin Class
class MyPhaserPlugin : public AEffect
{
public:
    MyPhaserPlugin();
    ~MyPhaserPlugin();

    // VST required functions
    virtual void processReplacing(float** inputs, float** outputs, VstInt32 sampleFrames);
    virtual VstInt32 processEvents(VstEvents* events);

private:
    // Private member variables and functions
    float m_frequency;
    float m_depth;
    float m_feedback;
    float m_wetDryMix;
    float m_phase;

    float applyPhaser(float inputSample);
};

MyPhaserPlugin::MyPhaserPlugin()
{
    // Initialize member variables and set default parameter values
    m_frequency = 0.5;
    m_depth = 0.5;
    m_feedback = 0.5;
    m_wetDryMix = 0.5;
    m_phase = 0.0;
}

MyPhaserPlugin::~MyPhaserPlugin()
{
    // Clean up resources
}

void MyPhaserPlugin::processReplacing(float** inputs, float** outputs, VstInt32 sampleFrames)
{
    // Loop through each sample in the input buffer
    for (int i = 0; i < sampleFrames; i++)
    {
        // Read the input sample and store it in a temporary variable
        float inputSample = inputs[0][i];

        // Apply the phaser effect to the input sample
        float outputSample = applyPhaser(inputSample);

        // Write the output sample to the output buffer
        outputs[0][i] = outputSample;
    }
}

VstInt32 MyPhaserPlugin::processEvents(VstEvents* events)
{
    // Handle MIDI events
    return 0;
}

float MyPhaserPlugin::applyPhaser(float inputSample)
{
    // Phase shift the input sample by the current phase offset
    float phaseShiftedSample = inputSample * cos(m_phase);

    // Apply feedback to the phase shifted sample and add it to the input sample
    float wetSample = (inputSample + phaseShiftedSample * m_feedback) * m_depth;

    // Mix the wet (phaser) signal with the dry (input) signal using the wet/dry mix ratio
    float outputSample = (wetSample * m_wetDryMix) + (inputSample * (1.0 - m_wetDryMix));

    // Update the phase offset for the next sample
    m_phase += m_frequency;

    // Return the output sample
    return outputSample;
}

// VST Plugin Export Function
extern "C" AEffect* VSTPluginMain(audioMasterCallback audioMaster)
{
    // Create a new instance of the plugin and pass it to the audioMaster
    return new MyPhaserPlugin();
}

