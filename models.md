---
layout: default
title: Models for Labinator
---

One of the principles of Labinator is that it had to be portable. While there are a lot of cheap/easy ways to meet this requirement, it certainly doesn't cause any harm to add a little flair or polish to everything. Besides... home labbing is an awesome hobby - so why not incorporate *more hobbies* into the process?

Enter: 3D modelling and 3D printing

During the pandemic I finally broke down and bought a 3D printer. I've always been interested in them and like many others, I had a bit more time on my hands than I knew what to do with! Having experimented with spline-based and polygonal 3D modeling in the past as well as playing around in Sketchup for years for other projects needing a twist of 3D, I dove in.

The below models were designed with the objectives of providing secure mounting for the portable lab, a modest amount of protection for the components while traveling, and the ability to be mounted/unmounted (possibly with some disassembly of Labinator) to replace components.

<script src="assets/js/modelviewer/stl_viewer.min.js"></script>
{% for model in site.static_files %}
    {% if model.path contains 'models' and model.path contains '.stl' %}
        <div class="modelinfo">
        <h3 class="modeltitle">{{ model.basename }}</h3>
        <!--
        <img src="images/{{ model.basename }}.jpg" />
        -->
        {% assign shortname=model.basename | slugify | replace: "-" "_" %}
        <progress id="pbtotal_{{ shortname }}" class="modelprogress" value="0" max="1"></progress>
        <div id="modelviewer_{{ shortname }}" class="modelviewer"></div>
        <script type="text/javascript">
            function load_prog_{{ shortname }}(load_status, load_session) {
            var loaded_{{ shortname }}=0;
            var total_{{ shortname }}=0;

            //go over all models that are/were loaded
            Object.keys(load_status).forEach(function(model_id) {
                //need to make sure we're on the last loading session (not counting previous loaded models)
                if (load_status[model_id].load_session==load_session) {
                loaded_{{ shortname }}+=load_status[model_id].loaded;
                total_{{ shortname }}+=load_status[model_id].total;
                }
            });

            //set total progress bar
            document.getElementById("pbtotal_{{ shortname }}").value=loaded_{{ shortname }}/total_{{ shortname }};
            }

            function load_done_{{ shortname }}() {
            document.getElementById("pbtotal_{{ shortname }}").style.display = 'none';
            document.getElementById("modelviewer_{{ shortname }}").style.display = 'unset';
            stl_viewer_{{ shortname }}.set_auto_rotate(true);
            }

            var setup_{{ shortname }} = {
            loading_progress_callback: load_prog_{{ shortname }},
            all_loaded_callback: load_done_{{ shortname }},
            models: [
                { 
                id: 0,
                filename: '../../../models/{{ model.name }}',
                color: '#d6d6d6',
                display: 'flat',
                rotationx: -1.5708,
                auto_rotate: true,
                }
            ]
            };

        console.log(setup_{{ shortname }}['models'][0]['filename']);
            var stl_viewer_{{ shortname }}=new StlViewer(document.getElementById("modelviewer_{{ shortname }}"), setup_{{ shortname }});
        </script>

        <ul>
            <li><a href="models/{{ model.basename }}.skp">Sketchup model</a></li>
            <li><a href="models/{{ model.basename }}.stl">STL model</a></li>
            <li><a href="modelview.html?model={{ model.name | url_encode }}">View it in 3D</a></li>
        </ul>
        </div>
    {% endif %}
{% endfor %}