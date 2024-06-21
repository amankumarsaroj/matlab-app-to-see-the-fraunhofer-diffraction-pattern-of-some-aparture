# matlab-app-to-see-the-fraunhofer-diffraction-pattern-of-some-aparture
classdef Diffraction_App < matlab.apps.AppBase

    % Properties that correspond to app components
    properties (Access = public)
        UIFigure                       matlab.ui.Figure
        ApertureDropDown               matlab.ui.control.DropDown
        ApertureDropDownLabel          matlab.ui.control.Label
        TextArea                       matlab.ui.control.TextArea
        AperturewidthEditField_2       matlab.ui.control.NumericEditField
        AperturewidthEditField_2Label  matlab.ui.control.Label
        Slider_2                       matlab.ui.control.Slider
        Slider                         matlab.ui.control.Slider
        GoButton                       matlab.ui.control.Button
        ShowapertureButton             matlab.ui.control.Button
        AperturewidthEditField         matlab.ui.control.NumericEditField
        AperturewidthEditFieldLabel    matlab.ui.control.Label
        ParametersLabel                matlab.ui.control.Label
        FraunhoferDiffractionPatternLabel  matlab.ui.control.Label
        ResultsAxes_3                  matlab.ui.control.UIAxes
        ResultsAxes_2                  matlab.ui.control.UIAxes
        ResultsAxes                    matlab.ui.control.UIAxes
        ApertureAxes                   matlab.ui.control.UIAxes
    end

    
    properties (Access = private)
        aperture %For different apertures
        M  %Number of values in meshgrid
    end
    
    methods (Access = private)
    end
    

    % Callbacks that handle component events
    methods (Access = private)

        % Code that executes after component creation
        function startupFcn(app)
            set(app.AperturewidthEditField_2, 'Enable', 'Off');
            set(app.AperturewidthEditField_2Label, 'Enable', 'Off');
            set(app.Slider_2, 'Enable', 'Off'); 
            set(app.AperturewidthEditField, 'Enable', 'Off');
            set(app.AperturewidthEditFieldLabel, 'Enable', 'Off');
            set(app.Slider, 'Enable', 'Off');  
            axis(app.ResultsAxes,'square');
            axis(app.ApertureAxes,'square');
        end

        % Button pushed function: ShowapertureButton
        function ShowapertureButtonPushed(app, event)
            app.M = 500;
            a = app.AperturewidthEditField.Value;
            b = app.AperturewidthEditField_2.Value;
            [x,y] = meshgrid(linspace(-1,1,app.M),linspace(-1,1,app.M));
            r = sqrt((x).^2 + (y).^2);
            if strcmp(app.ApertureDropDown.Value,'Slit')
                rect_y = abs(y) <= 1.00001;
                rect_x = abs(x) <= a;
                app.aperture = rect_x.*rect_y;
            elseif strcmp(app.ApertureDropDown.Value,'Rectangle')
                rect_y = abs(y) <= b;
                rect_x = abs(x) <= a;
                app.aperture = rect_x.*rect_y;
            elseif strcmp(app.ApertureDropDown.Value,'Circle')
                app.aperture = r <= a;
            elseif strcmp(app.ApertureDropDown.Value,'Triangle')
                app.aperture = (y >= (abs(x)-a)) .*  (y <=  0);
            elseif strcmp(app.ApertureDropDown.Value,'Double Slit')
                e = 2*a + b;
                rect_x = (abs(x) <= e/2) & (abs(x) >= b/2) ;
                rect_y = abs(y) <= 1.00001;
                app.aperture = rect_x .* rect_y;
            elseif strcmp(app.ApertureDropDown.Value,'Ring')
                r = sqrt((x).^2 + (y).^2);
                app.aperture = r <= b & r >= a;
            elseif strcmp(app.ApertureDropDown.Value,'Rhombus')
                a1 = (y <= (a-abs(x))) .*  (y >=  0);
                a2 = (y >= (abs(x)-a)) .*  (y <=  0);
                app.aperture = a1 | a2;
            elseif strcmp(app.ApertureDropDown.Value,'Star')
                a1 = (y <= (a-abs(x))) .*  (y >=  -a/2);
                a2 = (y >= (abs(x)-a)) .*  (y <=  a/2);
                app.aperture = a1 | a2;
            end
            imagesc(app.ApertureAxes,app.aperture);
            axis(app.ApertureAxes,'square');
            colormap(app.ApertureAxes,'hot');
            axis(app.ApertureAxes, 'tight');
        end

        % Button pushed function: GoButton
        function GoButtonPushed(app, event)
            U = fftshift(fft2(app.aperture));
            I = U .* conj(U);
            set(app.ResultsAxes,'YDir', 'Normal');
           
                imagesc(app.ResultsAxes,I);
                colormap (app.ResultsAxes,'turbo');
                axis(app.ResultsAxes,'square');
                axis(app.ResultsAxes,'tight');
                app.ResultsAxes.Title.String = 'Intensity Distribution';
            
                plot(app.ResultsAxes_2, I(app.M/2 + 1,:))
                app.ResultsAxes_2.Title.String = 'Intensity Curve';
           
                surf(app.ResultsAxes_3,abs(U));
                colormap (app.ResultsAxes_3, 'turbo');
                shading (app.ResultsAxes_3, 'interp');
                camlight (app.ResultsAxes_3, 'right');
                app.ResultsAxes_3.Title.String = '3D Plot';
           
             axis(app.ResultsAxes_3,'square');
             axis(app.ResultsAxes_3,'tight');
        end

        % Value changing function: Slider
        function SliderValueChanging(app, event)
            changingValue = event.Value;
            app.AperturewidthEditField.Value =  changingValue;
        end

        % Value changed function: AperturewidthEditField
        function AperturewidthEditFieldValueChanged(app, event)
            value = app.AperturewidthEditField.Value;
            app.Slider.Value = value;
        end

        % Value changed function: ApertureDropDown
        function ApertureDropDownValueChanged(app, event)
            value = app.ApertureDropDown.Value;
            if strcmp(value, 'Slit')
                set(app.AperturewidthEditField, 'Enable', 'On');
                set(app.AperturewidthEditFieldLabel, 'Enable', 'On');
                set(app.Slider, 'Enable', 'On'); 
                app.AperturewidthEditFieldLabel.Text = 'Slit width';
                set(app.AperturewidthEditField_2, 'Enable', 'Off');
                set(app.AperturewidthEditField_2Label, 'Enable', 'Off');
                set(app.Slider_2, 'Enable', 'Off');
            elseif strcmp(value, 'Rectangle')
                set(app.AperturewidthEditField, 'Enable', 'On');
                set(app.AperturewidthEditFieldLabel, 'Enable', 'On');
                set(app.Slider, 'Enable', 'On'); 
                set(app.AperturewidthEditField_2, 'Enable', 'On');
                set(app.AperturewidthEditField_2Label, 'Enable', 'On');
                set(app.Slider_2, 'Enable', 'On');
                app.AperturewidthEditFieldLabel.Text = 'Width';
                app.AperturewidthEditField_2Label.Text = 'Height';
            elseif strcmp(value, 'Circle')
                set(app.AperturewidthEditField, 'Enable', 'On');
                set(app.AperturewidthEditFieldLabel, 'Enable', 'On');
                set(app.Slider, 'Enable', 'On'); 
                set(app.AperturewidthEditField_2, 'Enable', 'Off');
                set(app.AperturewidthEditField_2Label, 'Enable', 'Off');
                set(app.Slider_2, 'Enable', 'Off');
                app.AperturewidthEditFieldLabel.Text = 'Radius';
            elseif strcmp(value, 'Triangle')
                set(app.AperturewidthEditField, 'Enable', 'On');
                set(app.AperturewidthEditFieldLabel, 'Enable', 'On');
                set(app.Slider, 'Enable', 'On'); 
                set(app.AperturewidthEditField_2, 'Enable', 'Off');
                set(app.AperturewidthEditField_2Label, 'Enable', 'Off');
                set(app.Slider_2, 'Enable', 'Off');
                app.AperturewidthEditFieldLabel.Text = 'Base width';
            elseif strcmp(value, 'Double Slit')
                set(app.AperturewidthEditField, 'Enable', 'On');
                set(app.AperturewidthEditFieldLabel, 'Enable', 'On');
                set(app.Slider, 'Enable', 'On'); 
                set(app.AperturewidthEditField_2, 'Enable', 'On');
                set(app.AperturewidthEditField_2Label, 'Enable', 'On');
                set(app.Slider_2, 'Enable', 'On');
                app.AperturewidthEditFieldLabel.Text = 'Slit width';
                app.AperturewidthEditField_2Label.Text = 'Slit distance';              
            elseif strcmp(value, 'Ring')
                set(app.AperturewidthEditField, 'Enable', 'On');
                set(app.AperturewidthEditFieldLabel, 'Enable', 'On');
                set(app.Slider, 'Enable', 'On'); 
                set(app.AperturewidthEditField_2, 'Enable', 'On');
                set(app.AperturewidthEditField_2Label, 'Enable', 'On');
                set(app.Slider_2, 'Enable', 'On');
                app.AperturewidthEditFieldLabel.Text = 'Inner Radius';
                app.AperturewidthEditField_2Label.Text = 'Outer Radius';
            elseif strcmp(value, 'Rhombus')
                set(app.AperturewidthEditField, 'Enable', 'On');
                set(app.AperturewidthEditFieldLabel, 'Enable', 'On');
                set(app.Slider, 'Enable', 'On'); 
                set(app.AperturewidthEditField_2, 'Enable', 'Off');
                set(app.AperturewidthEditField_2Label, 'Enable', 'Off');
                set(app.Slider_2, 'Enable', 'Off');
                app.AperturewidthEditFieldLabel.Text = 'Side width';
            elseif strcmp(value, 'Star')
                set(app.AperturewidthEditField, 'Enable', 'On');
                set(app.AperturewidthEditFieldLabel, 'Enable', 'On');
                set(app.Slider, 'Enable', 'On'); 
                app.AperturewidthEditFieldLabel.Text = 'Star width';
                set(app.AperturewidthEditField_2, 'Enable', 'Off');
                set(app.AperturewidthEditField_2Label, 'Enable', 'Off');
                set(app.Slider_2, 'Enable', 'Off');
            end
            
        end

        % Value changed function: AperturewidthEditField_2
        function AperturewidthEditField_2ValueChanged(app, event)
            value = app.AperturewidthEditField_2.Value;
            app.Slider_2.Value = value;
        end

        % Value changing function: Slider_2
        function Slider_2ValueChanging(app, event)
            changingValue = event.Value;
            app.AperturewidthEditField_2.Value = changingValue;
        end
    end

    % Component initialization
    methods (Access = private)

        % Create UIFigure and components
        function createComponents(app)

            % Create UIFigure and hide until all components are created
            app.UIFigure = uifigure('Visible', 'off');
            app.UIFigure.Color = [0.9412 0.9294 0.9412];
            app.UIFigure.Position = [100 100 758 507];
            app.UIFigure.Name = 'MATLAB App';

            % Create ApertureAxes
            app.ApertureAxes = uiaxes(app.UIFigure);
            title(app.ApertureAxes, 'Aperture')
            app.ApertureAxes.XTick = [];
            app.ApertureAxes.YTick = [];
            app.ApertureAxes.BoxStyle = 'full';
            app.ApertureAxes.LineWidth = 0.1;
            app.ApertureAxes.Box = 'on';
            app.ApertureAxes.Position = [230 248 261 193];

            % Create ResultsAxes
            app.ResultsAxes = uiaxes(app.UIFigure);
            title(app.ResultsAxes, 'Diffraction pattern')
            app.ResultsAxes.XTick = [];
            app.ResultsAxes.YTick = [];
            app.ResultsAxes.ClippingStyle = 'rectangle';
            app.ResultsAxes.Box = 'on';
            app.ResultsAxes.NextPlot = 'replaceall';
            app.ResultsAxes.Position = [230 44 254 192];

            % Create ResultsAxes_2
            app.ResultsAxes_2 = uiaxes(app.UIFigure);
            title(app.ResultsAxes_2, 'Line Profile')
            app.ResultsAxes_2.XTick = [];
            app.ResultsAxes_2.YTick = [];
            app.ResultsAxes_2.ClippingStyle = 'rectangle';
            app.ResultsAxes_2.Box = 'on';
            app.ResultsAxes_2.NextPlot = 'replaceall';
            app.ResultsAxes_2.Position = [483 249 254 192];

            % Create ResultsAxes_3
            app.ResultsAxes_3 = uiaxes(app.UIFigure);
            title(app.ResultsAxes_3, '3D plot')
            app.ResultsAxes_3.XTick = [];
            app.ResultsAxes_3.YTick = [];
            app.ResultsAxes_3.ClippingStyle = 'rectangle';
            app.ResultsAxes_3.Box = 'on';
            app.ResultsAxes_3.NextPlot = 'replaceall';
            app.ResultsAxes_3.Position = [483 44 254 192];

            % Create FraunhoferDiffractionPatternLabel
            app.FraunhoferDiffractionPatternLabel = uilabel(app.UIFigure);
            app.FraunhoferDiffractionPatternLabel.BackgroundColor = [0 0.502 0.502];
            app.FraunhoferDiffractionPatternLabel.HorizontalAlignment = 'center';
            app.FraunhoferDiffractionPatternLabel.FontSize = 18;
            app.FraunhoferDiffractionPatternLabel.FontWeight = 'bold';
            app.FraunhoferDiffractionPatternLabel.FontColor = [1 1 1];
            app.FraunhoferDiffractionPatternLabel.Position = [1 463 758 45];
            app.FraunhoferDiffractionPatternLabel.Text = 'Fraunhofer Diffraction Pattern';

            % Create ParametersLabel
            app.ParametersLabel = uilabel(app.UIFigure);
            app.ParametersLabel.FontWeight = 'bold';
            app.ParametersLabel.FontColor = [0.149 0.149 0.149];
            app.ParametersLabel.Position = [78 348 75 22];
            app.ParametersLabel.Text = 'Parameters';

            % Create AperturewidthEditFieldLabel
            app.AperturewidthEditFieldLabel = uilabel(app.UIFigure);
            app.AperturewidthEditFieldLabel.Position = [41 316 74 22];
            app.AperturewidthEditFieldLabel.Text = 'Parameter 1';

            % Create AperturewidthEditField
            app.AperturewidthEditField = uieditfield(app.UIFigure, 'numeric');
            app.AperturewidthEditField.Limits = [0 1];
            app.AperturewidthEditField.ValueChangedFcn = createCallbackFcn(app, @AperturewidthEditFieldValueChanged, true);
            app.AperturewidthEditField.Position = [149 316 33 22];
            app.AperturewidthEditField.Value = 0.1;

            % Create ShowapertureButton
            app.ShowapertureButton = uibutton(app.UIFigure, 'push');
            app.ShowapertureButton.ButtonPushedFcn = createCallbackFcn(app, @ShowapertureButtonPushed, true);
            app.ShowapertureButton.BackgroundColor = [0 0.502 0.502];
            app.ShowapertureButton.FontColor = [1 1 1];
            app.ShowapertureButton.Position = [68 108 95 44];
            app.ShowapertureButton.Text = 'Show aperture';

            % Create GoButton
            app.GoButton = uibutton(app.UIFigure, 'push');
            app.GoButton.ButtonPushedFcn = createCallbackFcn(app, @GoButtonPushed, true);
            app.GoButton.BackgroundColor = [0.0863 0.349 0.349];
            app.GoButton.FontColor = [1 1 1];
            app.GoButton.Position = [89 58 53 40];
            app.GoButton.Text = 'Go';

            % Create Slider
            app.Slider = uislider(app.UIFigure);
            app.Slider.Limits = [0 1];
            app.Slider.ValueChangingFcn = createCallbackFcn(app, @SliderValueChanging, true);
            app.Slider.Position = [47 302 137 3];
            app.Slider.Value = 0.1;

            % Create Slider_2
            app.Slider_2 = uislider(app.UIFigure);
            app.Slider_2.Limits = [0 1];
            app.Slider_2.ValueChangingFcn = createCallbackFcn(app, @Slider_2ValueChanging, true);
            app.Slider_2.Position = [47 215 137 3];
            app.Slider_2.Value = 0.1;

            % Create AperturewidthEditField_2Label
            app.AperturewidthEditField_2Label = uilabel(app.UIFigure);
            app.AperturewidthEditField_2Label.Position = [40 235 74 22];
            app.AperturewidthEditField_2Label.Text = 'Parameter 2';

            % Create AperturewidthEditField_2
            app.AperturewidthEditField_2 = uieditfield(app.UIFigure, 'numeric');
            app.AperturewidthEditField_2.Limits = [0 1];
            app.AperturewidthEditField_2.ValueChangedFcn = createCallbackFcn(app, @AperturewidthEditField_2ValueChanged, true);
            app.AperturewidthEditField_2.Position = [152 235 33 22];
            app.AperturewidthEditField_2.Value = 0.1;

            % Create TextArea
            app.TextArea = uitextarea(app.UIFigure);
            app.TextArea.HorizontalAlignment = 'right';
            app.TextArea.FontColor = [1 1 1];
            app.TextArea.BackgroundColor = [0.1608 0.4588 0.4353];
            app.TextArea.Position = [1 5 758 24];
            app.TextArea.Value = {'MATLAB Assignment - An application by Anshika and Aman'};

            % Create ApertureDropDownLabel
            app.ApertureDropDownLabel = uilabel(app.UIFigure);
            app.ApertureDropDownLabel.Position = [40 398 74 22];
            app.ApertureDropDownLabel.Text = 'Aperture';

            % Create ApertureDropDown
            app.ApertureDropDown = uidropdown(app.UIFigure);
            app.ApertureDropDown.Items = {'No aperture', 'Slit', 'Rectangle', 'Circle', 'Triangle', 'Double Slit', 'Ring', 'Rhombus', 'Star'};
            app.ApertureDropDown.ValueChangedFcn = createCallbackFcn(app, @ApertureDropDownValueChanged, true);
            app.ApertureDropDown.BackgroundColor = [1 1 1];
            app.ApertureDropDown.Position = [104 398 81 22];
            app.ApertureDropDown.Value = 'No aperture';

            % Show the figure after all components are created
            app.UIFigure.Visible = 'on';
        end
    end

    % App creation and deletion
    methods (Access = public)

        % Construct app
        function app = Diffraction_App

            % Create UIFigure and components
            createComponents(app)

            % Register the app with App Designer
            registerApp(app, app.UIFigure)

            % Execute the startup function
            runStartupFcn(app, @startupFcn)

            if nargout == 0
                clear app
            end
        end

        % Code that executes before app deletion
        function delete(app)

            % Delete UIFigure when app is deleted
            delete(app.UIFigure)
        end
    end
end
